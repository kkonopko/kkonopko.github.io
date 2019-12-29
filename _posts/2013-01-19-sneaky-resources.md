---
layout: post
title: Sneaky resources
date: 2013-01-19 15:45
---

Every so often an application has to deal with some sort of resources. There's
been plenty of books and articles written about it and they are all useful. They
even attempt to go beyond managing memory as this is only one of many
resources. The resources often mentioned are network connections, database
connections, file system objects and the like. Not only the application has to
release them when it's done with them but also has to ensure that they are not
held when errors occur and a different execution path is taken.

## What's the fuss all about?

In C++ error conditions are very often signalled by throwing an exception which
automatically unwinds the call stack. This is very handy if one knows how to
leverage it. For unwary this could be pernicious though. In some ways C is more
convenient in this regard as there could not be any exception thrown when a C
function is called. The real problem I often see in code is when both worlds
have to coexist:

* C++ -> C call

  The most likely scenario is when a C++ code calls some C functions that return
  some sort of handles (very often pointers) to some resources (or simply memory)
  which are expected to be released with another (complementary) C function call.

* C -> C++ call

  The other scenario is when a C code calls some C++ function (quite often some
  sort of C++ callback function registered with a C API). Care need to be taken to
  not let any exceptions back into C as this causes undefined behaviour. If you're
  lucky the program aborts immediately, otherwise you have to gape at meaningless
  call stacks and waste some time to figure out what happened. In this situation
  (if you know that C calls C++) a good idea is to set a breakpoint on a library
  routine that is responsible for delivering exceptions (for example `__cxa_throw`
  in glibc) and laboriously debug the program to find which exception gets into C.

In this article I want to focus on the C++ calling C scenario.

Most of resources are released by the operating system if the application exits
or terminates. So network connections, file descriptors and all sort of sockets
are closed, memory is released etc. There are some less obvious resources though
which result in leaving undeleted temporary files behind, unreleased IPC
resources or (for paranoid) unwiped memory (RSA encryption/decryption). I admit
I haven't ever attempted to deal with application recovery and I'm usually happy
if the application at least reports some error message and simply exits. But in
the face of a less obvious resources leakage and simply for diligence I like to
have all toys tidied up nicely no matter what bad course the application has
taken. This does not include an immediate abortion which is really horrible and
there's not much one can do about it. Even if releasing resources on application
exit doesn't matter now, some bits of code or the entire application may get
reused somewhere where it matters.


Some unwary programmer may write something like this:

{% highlight C++ %}
void
foo()
{
  char *const p = get_some_c_string();
  std::string s(p);
  // do some fancy stuff with the string
  free(p);
}
{% endhighlight %}

Now this is a typical scenario when talking about exceptions. In this situation
a lot of operations with `std::string` (including its creation) can throw. This
will inevitably leak memory returned by `get_some_c_string()`. I won't dwell on it
as a lot has been said about it elsewhere and it usually ends up using some sort
of smart pointers.

## What to do? What to do?

Let me introduce some small utility function that creates a scoped pointer that
draws upon `std::unique_ptr` from C++11.

{% highlight C++ %}
template<class T, class D = std::default_delete<T>>
constexpr std::unique_ptr<T, D>
make_scoped(T *const ptr, D deleter = D())
{
  return std::unique_ptr<T, D>(ptr, deleter);
}
{% endhighlight %}

Now let's assume that one wants to verify a certificate using OpenSSL. This task
itself could probably make a lot of people nervous not to mention they would
have to manage related resources properly. Now let's focus on adding some
policies to the verification process:

{% highlight C++ %}
void
addPolicy(X509_VERIFY_PARAM *const params, const std::string& policy)
{
  auto policyObj =
    make_scoped(OBJ_txt2obj(policy.c_str(), 1), ASN1_OBJECT_free);
    
  if (!policyObj) {
    throw std::runtime_error("Cannot create policy object");
  }

  if (!X509_VERIFY_PARAM_add0_policy(params, policyObj.get())) {
    throw std::runtime_error("Cannot add policy");
  }

  // committed
  (void)policyObj.release();
}
{% endhighlight %}

An interesting use case here is a "transactional" approach. Note that OpenSSL
functions that have *0* in their name take ownership of objects passed to
them. This is explained in the notes section
[here](https://www.openssl.org/docs/crypto/crypto.html) but it may appear to you
to be the opposite if you read it for the first time. The `addPolicy()` function
creates a managed policy object and do not hesitate to throw if something goes
wrong. But once we successfully passed the ownership of the policy object to
another object, we can give up its ownership in our scope (remember that C
functions do not throw) and the "transaction" is "committed". Nice and
easy. Then some part of initialization of the verification process could look
like this:

{% highlight C++ %}
const auto params =
  make_scoped(X509_VERIFY_PARAM_new(), X509_VERIFY_PARAM_free);
  
(void)X509_VERIFY_PARAM_set_flags(
  params.get(), X509_V_FLAG_POLICY_CHECK | X509_V_FLAG_EXPLICIT_POLICY);
(void)X509_VERIFY_PARAM_clear_flags(
  params.get(), X509_V_FLAG_INHIBIT_ANY | X509_V_FLAG_INHIBIT_MAP);
  
addPolicy(params.get(), "1.2.3.4");

// set up other stuff and do the verification
// ...
{% endhighlight %}

This is even nicer. It's a bliss and harmony.

I mentioned that letting the program exit without releasing memory shouldn't
make any harm as the operating system would reclaim it anyway. But if you're
paranoid and deal with some secrets in memory, then this becomes an issue. To
address this problem just do the following:

{% highlight C++ %}
void
someRsaStuff()
{
  auto rsa = make_scoped(RSA_new(), RSA_free);
  // do some stuff with the RSA key
  // ...
}
{% endhighlight %}

Note that [`RSA_free()`](https://www.openssl.org/docs/crypto/RSA_new.html)
function erases the memory.

It's also convenient to do some more complicated things with lambdas which in
pre-C++11 would have to be wrapped into a functor:

{% highlight C++ %}
const auto untrustedCerts =
  make_scoped(
    sk_X509_new_null(),
    [](STACK_OF(X509)* const s) { sk_X509_pop_free(s, X509_free); });
// now add untrusted certificates to the stack
// ...
{% endhighlight %}

## No worries

Using some C++11 goodies can help you to manage different sort of resources and
focus on the problem rather worrying about releasing stuff especially in error
paths. It also gives a confidence and clarity of what is released where and
how. In some scenarios you can also use the transactional approach.


Source code for the examples above is available
[here](https://github.com/kkonopko/kriscience/blob/master/scoped-ptr-example/scoped-ptr-example.cpp). On
my Fedora 17 I built it as follows:

{% highlight terminal %}
g++ -std=c++11 scoped-ptr-example.cpp -lcrypto -o /tmp/scoped
{% endhighlight %}

For those who can't afford using C++11 but can use [Boost
libraries](http://www.boost.org/), the following could be a replacement for a
scoped pointer with a custom deleter:

{% highlight C++ %}
#include <boost/function.hpp>
#include <boost/interprocess/smart_ptr/unique_ptr.hpp>

namespace detail {

template<class T>
class unique_ptr_deleter
{
public:
    template<class D>
    unique_ptr_deleter(const D& d) : deleter(d) {}

    void operator()(T* const p) throw() { deleter(p); }

private:
    const boost::function<void (T* const)> deleter;
};

} // namespace detail

template<class T>
struct unique_ptr
{
    typedef boost::interprocess::unique_ptr<
      T, detail::unique_ptr_deleter<T> > type;
};
{% endhighlight %}

This can be used as follows:

{% highlight C++ %}
const unique_ptr<RSA>::type rsa(RSA_new(), RSA_free);
{% endhighlight %}

{% capture subject %}
{{ page.title }}
{% endcapture %}
{% include comments.html title=subject %}
