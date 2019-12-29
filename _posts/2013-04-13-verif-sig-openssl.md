---
layout: post
title: Verifying signatures with OpenSSL API
date: 2013-04-13 20:19
---

This is another article where I try to tame OpenSSL API using C++11. This time
round I describe a small example showing how to verify signed data
programatically. There are many message formats catering for different needs. In
this example I show how to verify that the data is not tampered and is sent from
a party identified by a PKI certificate. Please refer to my other article to
learn how to verify a certificate.

## A brief introduction

Again I want to emphasize that you should not implement this functionality as
you can use `openssl` tool:

{% highlight terminal %}
openssl dgst -verify test-key-pub.pem \
  -signature /tmp/signature </tmp/data
{% endhighlight %}

This command checks that the data stored in `/tmp/data` is not tampered. The
tool calculates a checksum (a digest) and verifies it with the signature stored
in `/tmp/signature`. The signature has been signed with the private key paired
with the public key stored in test-key-pub.pem. If there's a certificate
associated with the public key available, it can also be verified to see whether
the data hasn't been signed by an intruder in the middle.

As you can see, there's no need to invent the wheel if your requirements are
simple enough. Depending on the circumstances this approach might not be
sufficient or acceptable, and only then you should come to grips with your own
implementation.

Producing or verifying a signature is rather expensive operation as it involves
asymmetric cryptography. In practice a digest is produced first (e.g. using
SHA1) and then the digest is signed with one of the asymmetric keys. The
verification comprises applying the same digest function to the received data
and checking whether the signature of that digest "matches" when using the other
key of the asymmetric pair. You don't have to worry about these details though
as they are hidden behind the OpenSSL API. Hopefully this also allays concerns
about the use of the `openss dgst` command which stands for "digest". The
signature is simply another step in the process of digesting data.

## Code

Code for this example is available
[here](https://github.com/kkonopko/kriscience/blob/master/signature-verify/signature-verify.cpp). There's
also a very basic [test script](https://github.com/kkonopko/kriscience/blob/master/signature-verify/signature-verify-test.sh) provided.

The main three functions we are going to use are
[`EVP_VerifyInit_ex()`](https://www.openssl.org/docs/crypto/EVP_VerifyInit.html),
[`EVP_VerifyUpdate()`](https://www.openssl.org/docs/crypto/EVP_VerifyInit.html),
and
[`EVP_VerifyFinal()`](https://www.openssl.org/docs/crypto/EVP_VerifyInit.html). The
first two of them are simply aliases (macros) of equivalent "digest"
functions. Of course you shouldn't abuse them and better use the macros provided
to be explicit about the intentions. Please also note that in general `*_ex()`
versions of OpenSSL API functions are recommended if available as they are more
general and allow you to use an engine. If you don't intend to use an engine
simply use `nullptr`.

First you initialise the algorithm, then there's a one or more updates that feed
the algorithm with data, and in the end you finalise the algorithm. The update
step allows to process "streamed" data, i.e. you feed the algorithm with data as
it arrives. If all data is available at once, you can make only one update
call. In many situation though you might want to process data in chunks,
e.g. when you read a large file or from a network socket.

More details about the API used in this example are available in the
[manual](https://www.openssl.org/docs/crypto/EVP_VerifyInit.html) so there's no
point in duplicating them here. If you're off-line and have openssl-devel (or
equivalent) package installed (which you should in order to compile this
example), you can also use info or man pages. Don't forget to read about
[`EVP_MD_CTX_create()`](https://www.openssl.org/docs/crypto/EVP_DigestInit.html)
and [`EVP_MD_CTX_destroy()`](https://www.openssl.org/docs/crypto/EVP_DigestInit.html).

## Build and test

This is how I build the example on my Fedora 18 laptop:

{% highlight terminal %}
g++ -std=c++11 -O3 -DNDEBUG signature-verify.cpp \
  -lcrypto -o /tmp/my-verifier
{% endhighlight %}

I think that the most frustrating thing about keys, certificates and all this
cryptographic stuff is testing. Creating test assets (key material, certificates
etc.) can be truly onerous. But this is still not as hard as testing a full
production system with real cryptographic material (very often hardware
assisted), so let's get on with it:

{% highlight terminal %}
# generate test private key and associated certificate
openssl req -x509 -newkey rsa:2048 \
  -keyout test-key-priv.pem -subj "/CN=FakeSigner" \
  -passout pass:none -out test-cert.pem

# sign some test data
echo -n "test" | tee /tmp/data | \
openssl dgst -sha1 -sign test-key-priv.pem \
  -passin pass:none -out signature
{% endhighlight %}

And finally we can run our verifier:

{% highlight terminal %}
# verify signed data
/tmp/my-verifier test-cert.pem /tmp/data signature
{% endhighlight %}

As the steps above are a bit tedious, you can use a test script I provide
here. Simply give it the path to the verifier executable as an argument and
that's it. It creates a temporary scratch directory where it generates the
assets and runs rudimentary tests using the executable provided. As I wanted to
keep it dead simple, it doesn't provide any additional options like preserving
the scratch directory, setting verbosity level etc. It'll probably evolve in
future incarnations once I've got examples in my [repository](https://github.com/kkonopko/kriscience) a bit reorganised.

{% highlight terminal %}
./signature-verify-test.sh /tmp/my-verifier
{% endhighlight %}

Happy verifying!

{% capture subject %}
{{ page.title }}
{% endcapture %}
{% include comments.html title=subject %}
