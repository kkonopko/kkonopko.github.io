---
layout: my-post
title: Supporting trusted but untrusted certificates with OpenSSL
date: 2013-03-02 16:19
---

This time for something completely different I'll broach a bit intimidating
area—PKI certificate chains that link back both to trusted and untrusted root
certificates. That is, how to recognise different trees from quite a long way
away.

For some of the readers that know at least a little bit about the matter this
might be a quick and easy recipe to solve their problem if they understand
it. For those who have no idea of what I'm talking about this still might be an
interesting reading if they like exotic trips. Don't worry though, as I'll
provide some basic introduction without going into much detail. There's already
a smorgasbord of material to learn from if someone's interested.

I won't even bother providing links as a reference (except for salient ones) as
I find it overwhelming and distracting when a very specific story is sprinkled
with lots of links for everything. Sure, one may ignore them (so I'd waste my
time providing them) but equally one may look up something on their own if they
have an urge to do so (whatever you look for, if you need an authoritative
information, always reach out for RFCs). This is what I find to be a pragmatic
approach which I think is different from purely scientific one.

## A bit of introduction

_PKI_ stands for _Public Key Infrastructure_ and is a monster which a lot of
people find hard to get on with. For this article it's only important to know
that it provides means of distributing some cryptographic material referred to
as public key in a trusted manner.


What is a public key you may ask and why is it public? Some smart guys in the
past have combined a bunch of mathematical operations with huge random numbers
and have come up with something known as asymmetric keys. Keys in turn are
specially crafted strings of digits used as an input for ciphers. So you have:
data + key -> cipher => blob (a 'cipher' is a function that eats a 'data' and a
'key' and spews out a 'blob'). Asymmetric means that there's a pair of keys
related to each other through some mathematical operations—you can use one to
encipher data and the second one to decipher it and the other way around. If you
keep one of them secret and give the other one to everyone else you have
public-private key pair.

## Publishing keys

When you give out your public key, someone may want to use it to encipher
something that is only intended for you (some secret) as you are supposed to use
your private key to decipher it. The problem is that by simply publishing your
public key there's no guarantee that it's actually your key. No one can really
trust it to encipher secrets they want to share only with you. Here is where
certificates come into play.

Certificates leverage at least two properties of asymmetric ciphers:
authenticity and integrity of data. Again, using some smart mathematical
operations and having one of the asymmetric keys, one can tell that the data has
not been tampered with and has been ciphered with the other key of the same
pair. This process is known as signing and verification and the data exchanged
between parties is called a signature.

Certificates are structured bits of information where the essential part is a
signature of one party's (A) public key created with a more trusted party's (B)
private key. Such signature in turn can be verified with a more trusted party's
(B) public key which again can be signed by a more trusted party's (C) private
key (another certificate). This forms a chain of trust (chain of certificates)
which ends up at the top with a root certificate.

## Root certificates

Root certificates are special as there's no one who would signed them. They are
in fact self-signed. So how does it all make sense? At the very top is a
human. It might be you or a system administrator (some people claim they are not
humans). No matter how complex processes and policies are employed, it's always
a human being that makes the ultimate decision: I trust something or not.

## Root certificate bundles

A system that is supposed to use PKI needs a set of trusted certificates also
known as a root CA certificate store (CA stands for _Certificate Authority_) or
a root CA certificate bundle (CA bundle for short). Whatever chain of
certificates it needs to verify, it expects that the top-level untrusted
certificate in that chain is signed with one of the root certificates stored in
the root CA bundle which it ultimately trusts. As a PC user you get a CA bundle
onto your system usually distributed with a web browser so you can use HTTPS
protocol. You might have been prompted by a browser with some sort of security
exception pop-up when visiting a website using HTTPS protocol whose authenticity
couldn't be verified by the browser based on the CA bundle installed (actually
the certificate chain sent to your browser as part of the TLS protocol could not
be verified or for example the website domain name did not match the one signed
by the leaf certificate). The browser in such situation might leave the decision
up to you whether to trust the website or not.

## Summary

This short introduction just scratched the surface. It's not even a top of an
iceberg. There's a lot more to talk about, like on what basis a human being can
make a decision to trust an authority (or simply someone else's public key) or
how to ensure key privacy, what different key usages are and how they are
ensured and enforced, cipher suites properties etc. Firstly, I'm not an expert
nor a scientist, and secondly, it's not directly related to the rest of this
article.

## What this article is actually about?

Given the introduction above or the knowledge you might have already had and a
CA bundle, you might face a situation when you need to verify a certificate
chain as follows: A signed by B signed by C signed by D but C is in your CA
bundle (your system trusts it ultimately) and D is not (your system doesn't
trust it). I'm pretty sure it happened at least once in everyone's life, even in
my dog's life which I actually don't have.

You may wonder how it's possible to have certificate C in the bundle
(self-signed) and at the same time have it signed with certificate D. Here's
where I'd like to refer to some external
[resource](http://www.confusedamused.com/notebook/fixing-verisign-certificates-on-windows-servers)
where it's nicely depicted. Note that some people may burble here something
about cross-signing but it's not specific only to this situation as
cross-signing simply refers to anything else than self-signing. Don't get
confused by the diagram on the website I referred to. The article there says
that one certificate is signed by two other certificates. But it's _not_ a
certificate that is signed nor any certificate signs another one. It's _keys_
what actually gets signed. So by saying that certificate B signs certificate A,
one (hopefully) means that the public key represented by certificate A is signed
by the private key corresponding to the public key _represented_ by certificate
B.

## What's the catch?

The catch is that if use OpenSSL, you may not be able to verify a chain which
contains a root certificate included in the system CA bundle but signed in that
chain by some other non-trusted root certificate. This is not what is expected
in many situations. Fear not my friend as you're not left alone. OpenSSL has a
solution for this situation: `X509_V_FLAG_TRUSTED_FIRST` verification flag. It
tells it to not follow the chain once a trusted certificate is found.

Now the problem is that it's not available in all OpenSSL versions. The way I
understand OpenSSL releases is that at the time of this writing there are three
"production" branches available: 0.9.8x, 1.0.0x and 1.0.1x. You're very likely
using one of them. None of them supports `X509_V_FLAG_TRUSTED_FIRST` though. All
you need to do is to apply the following
[patch](http://git.openssl.org/gitweb/?p=openssl.git;a=commitdiff;h=db28aa86e00b9121bee94d1e65506bf22d5ca6e3)
from the OpenSSL mainline. Now given a naughty certificate chain in `chain.pem`
and a CA bundle in `ca-bundle.crt` you may try:

{% highlight terminal %}
[kris@lenovo-x1 kriscience]$ openssl verify \
-CAfile ca-bundle.crt -untrusted chain.pem chain.pem

chain.pem: C = US, O = "VeriSign, Inc.", OU = VeriSign Trust Network, OU = "(c) \
2006 VeriSign, Inc. - For authorized use only", CN = VeriSign Class 3 Public \
Primary Certification Authority - G5
error 20 at 2 depth lookup:unable to get local issuer certificate
{% endhighlight %}

but if you use a new `-trusted_first` option, it should succeed:

{% highlight terminal %}
[kris@lenovo-x1 kriscience]$ openssl verify \
-CAfile ca-bundle.crt -trusted_first -untrusted chain.pem chain.pem
cert.pem: OK
{% endhighlight %}

Now all you need to do is to convince your client application to use
`X509_V_FLAG_TRUSTED_FIRST` option. For example if you are using libcurl, you
may want to apply this patch:

{% highlight diff %}
--- a/lib/ssluse.c
+++ b/lib/ssluse.c
@@ -1651,6 +1651,26 @@ ossl_connect_step1(struct connectdata *conn,
           data->set.str[STRING_SSL_CRLFILE]: "none");
   }
 
+  if (1) { // artificial scope to keep the patch local
+    X509_VERIFY_PARAM *x509_param = X509_VERIFY_PARAM_new();
+      if (!x509_param) {
+          failf(data,"failed to create X509 verification parameter");
+          return CURLE_OUT_OF_MEMORY;
+      }
+
+      if (!X509_VERIFY_PARAM_set_flags(x509_param, X509_V_FLAG_TRUSTED_FIRST)) {
+          failf(data,"failed to set X509 flag - trusted certificate first");
+          return CURLE_PEER_FAILED_VERIFICATION;
+      }
+
+      if (!SSL_CTX_set1_param(connssl->ctx, x509_param)) {
+          failf(data,"failed to set X509 trusted certificate first parameter");
+          return CURLE_PEER_FAILED_VERIFICATION;
+      }
+
+      X509_VERIFY_PARAM_free(x509_param);
+  }
+  
   /* SSL always tries to verify the peer, this only says whether it should
    * fail to connect if the verification fails, or if it should continue
    * anyway. In the latter case the result of the verification is checked with
{% endhighlight %}

## Notes

A nice OpenSSL PKI tutorial: [https://pki-tutorial.readthedocs.org/en/latest](https://pki-tutorial.readthedocs.org/en/latest)
