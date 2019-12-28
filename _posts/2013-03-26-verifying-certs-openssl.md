---
layout: post
title: Verifying certificates with OpenSSL API
date: 2013-03-26 21:30
---

I'd like to allude to my [previous article]({% post_url 2013-03-02-certs-openssl%}) where I superficially described how certificates and
PKI work, at least the way I understand it. This time I want it to be more
tangible and hope to make it helpful to anyone grappling with a problem: how to
verify certificates programatically with OpenSSL API.

Let me make it clear—don't use OpenSSL API, don't write any C/C++ or any other
application. Check out ready to use tool, namely `openssl verify` command:

{% highlight terminal %}
[kris@lenovo-x1 tmp]$ cat c1.pem c2.pem c3.pem > untrusted.pem
[kris@lenovo-x1 tmp]$ openssl verify \
  -CAfile ca.pem \
  -untrusted untrusted.pem \
  leaf.pem 
leaf.pem: OK
{% endhighlight %}

This should cover most common cases. Sure, one may say this is not a production
tool and despite been written by experts (the authors of the OpenSSL package)
it's hard to audit or it doesn't provide required features from the command
line. Someone may not be able to deploy the executable onto the target system
due to space limitations or may not be able to execute it from their
program. There are plenty of reasons why someone would want to use OpenSSL API
directly. But if you're not constrained by any of those reasons, just don't,
unless you want to learn something or do it for dubious fun. And the OpenSSL API
is not the most beautiful one. See [this article](https://threatpost.com/en_us/blogs/ssl-vulnerabilities-found-critical-non-browser-software-packages-102512)
on how botched SSL APIs make it easy to introduce vulnerabilities into
applications.

I doubt you can get the ball rolling after reading the rather sloppy and
incomplete documentation. Maybe it's a good starting point but sooner or later
you'll be discontented. The way I learned how to use the API—and I believe it's
the best way—is to read OpenSSL source code and maybe run the harder bits under
the debugger. Certificate verification in particular is implemented in
`apps/verify.c`. A good code cross-reference tool might be handy as well as any
grep-ish approach will be hindered by wodge of preprocessor generated functions.


It's also interesting to see how C++11 features can be leveraged to deliver
robust and nice to read code. Some developers slog through C in their
applications as they feel they have to because OpenSSL is written in C. Sure, I
appreciate OpenSSL itself is written in C as it's portable, but writing robust
code in C is _really_ hard and unpleasant. I wouldn't bother writing an example
for this article in C at my leisure. I wouldn't even do it for money if there
was no good reason for it. With C++11 on the other hand it's fun!

This is a rudimentary example and there are no classes and all that fancy
stuff. I wanted to keep it dead simple. You may wonder why I bother checking all
error codes and throw everywhere. Well, the reason is simple—I never know when I
want to copy it (ehem, re-factor) into some utility, class, library or anything
else in production code. I strongly believe that it's very important to start
with a prototype (the simpler the better) to explore new areas and spot problems
as soon as possible. But the sad thing is that quite often prototype code ends
up in a final release, most notably because "if it works, don't touch it". I see
nothing wrong in reusing prototype code that proved to work really well. But the
caveat is to make it robust from the outset. So all this "small" things like
checking error codes, deallocating those tiny bits and pieces may not matter
now, but they might do in the future. And you don't want to go through the code
scouring it and looking for these little nuisances. It's much easier to deal
with the details as you write the "essential" code.

Enough preaching so let's do it, shall we? The source code is available as usual
on GitHub [here](https://github.com/kkonopko/kriscience/blob/master/cert-verify/cert-verify.cpp).

I want to declare some function return types with my beloved
`std::unique_ptr`. As this is a very lightweight smart pointer, the pointee
deleter is part of its type. This is different when compared to
`std::shared_ptr` which uses type erasure for the deleter. As I don't care too
much about the performance here, I want to do the same for my `std::unique_ptr`:

{% highlight C++ %}
template<class T>
using scoped_ptr = std::unique_ptr<T, std::function<void (T *const)>>;
{% endhighlight %}

And I also reuse a teensy `make_scoped()` utility from one of the [previous
articles]({% post_url 2013-01-19-sneaky-resources %}). It's noteworthy to say
that this might be considered as `std::unique_ptr` abuse as it just happens that
objects returned by OpenSSL API are literary pointers. Had they been some sort
of handles only, I wouldn't have used `std::unique_ptr` here. And I named these
utilities `scoped_ptr` and `make_scoped` to make them different from proposed
`std::make_unique()`.

Before you can call
[`X509_verify_cert()`](https://www.openssl.org/docs/crypto/X509_verify_cert.html),
you need to prepare certificate store context which comprises ultimately trusted
root certificate, untrusted intermediate certificates and the leaf
certificate. This is all done in the `main()` function and is fairly easy to
follow. Quite challenging is providing the actual error message. In this example
I limited it to loading some rather succinct message from the OpenSSL library
and extracting certificate expiry dates as I've found it quite common
verification failure reason (dealing with a lot of generated short-term test
certificates). See the formidable `print_verification_failure_msg()`. Basically
OpenSSL provides verification algorithm but you could provide your own. Believe
me though—you don't want to do it. So here's a simplified sequence (please refer
to the [source code](https://github.com/kkonopko/kriscience/blob/master/cert-verify/cert-verify.cpp)
for detailed error handling):

{% highlight C++ %}
// Let's create a certificate store for the root CA
const auto trusted =
  make_scoped (X509_STORE_new (), X509_STORE_free);

// The lookup method is owned by the store once created
// and added to the store
const auto lookup =
  X509_STORE_add_lookup (trusted.get (), X509_LOOKUP_file ());

// Load the root CA into the store
X509_LOOKUP_load_file (lookup, argv[1], X509_FILETYPE_PEM);

// Create a X509 store context required for the verification
const auto ctx =
  make_scoped (X509_STORE_CTX_new (), X509_STORE_CTX_free);

// Now our untrusted (intermediate) certificates (if any)
const auto untrusted =
  read_untrusted (argv + 2, argv + argc - 1);

// And our leaf certificate we want to verify
const auto cert =
  read_x509 (argv[argc - 1]);

// Initialize the context for the verifiacion
X509_STORE_CTX_init (
  ctx.get (), trusted.get (), cert.get (), untrusted.get ());

// Verify!
const int result = X509_verify_cert (ctx.get ());
{% endhighlight %}

To build and try the example I did the following on my Fedora 18 laptop:

{% highlight terminal %}
[kris@lenovo-x1 kriscience]$ g++ -std=c++11 -O2 -DNDEBUG \
cert-verify.cpp -lssl -lcrypto -o /tmp/test
[kris@lenovo-x1 kriscience]$ !$ ca.pem c1.pem c2.pem c3.pem leaf.pem
Verification OK
{% endhighlight %}

This is very trivial example but it's a foothold for someone starting their
adventure with OpenSSL API. A more representable application would use policy
verification, time parameter and CRL checks. Maybe one day I'll present it here.
