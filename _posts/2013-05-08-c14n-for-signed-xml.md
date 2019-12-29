---
layout: post
title: C14N for signed XML
date: 2013-05-08 23:29
---

Here's another, somewhat exotic subject which is
[canonicalisation](https://en.wikipedia.org/wiki/Canonicalization), quite often
abbreviated C14N. No, this has nothing to do with the pope and saints. This is
about getting the canonical form of an XML file.

## Confession

Why? Well, XML is pretty ambiguous format where there's a lot of white space
characters (for poor human readability), no strict element ordering or implicit
namespaces. So what? This means that for a dumb machine two identically looking
XML files may be very different. An extreme example would be a terminating new
line character which is very difficult to spot. So what? Sometimes you want to
find out if two XML files are identical. Or even more—whether anyone has
tampered with it and whether it's been sent by someone you expect if it's
transferred over the internet. This all leads to a state of the art, 21st
century invention—[signed XML or
XMLDSig](https://en.wikipedia.org/wiki/XML_Signature).

XML files are not perfect but some people love them. This is not going to be an
overview of this format as this is a very broad topic and I'm not eligible to
write such overview. I'm just a lowly engineer. Let me just say that XML is
useful sometimes. I'd even dare to say that it's inevitable to use sometimes (as
a dead simple INI format would raise too many eye-browses... OK, I won't be
sarcastic any more). XML is inevitable sometimes. So I convinced you and you
think that signed XML is cool and you want to have one, aye? Your mates will be
jealous. You may be even tempted to get your partner's name (or your pet's name
if you don't have a partner or anyone or anything you're attached to) into XML
form, sign it and tattoo on your thigh. I'll leave it to you and focus only on
one mysterious step toward that goal which is canonicalisation.

Since you're still reading, you're either one of my committed readers or you're
genuinely interested in XMLDSig. I won't recommend reading [XMLDSig reference](http://www.w3.org/TR/xmldsig-core/) nor
[C14N](http://www.w3.org/TR/xml-c14n) unless you have a strong urge to wade through rather dry standards. I also
won't recommend implementing anything from scratch. Check out [xmlsec](http://www.aleksey.com/xmlsec/) library
whether it suits you. I confess I knew about _xmlsec_ but decided to use bare
[libxml2](http://www.xmlsoft.org/) and get my hands dirty for at least two reasons—I was scared and I
wasn't sure what I was doing. I wanted to reassure myself by having as much
control as possible. I hope you're in a better position.

## Not very discerning dissection

Briefly XMLDSig consists of three parts:

* [SignedInfo](http://www.w3.org/TR/xmldsig-core/#sec-SignedInfo)
* [KeyInfo](http://www.w3.org/TR/xmldsig-core/#sec-KeyInfo)
* [Object](http://www.w3.org/TR/xmldsig-core/#sec-Object)

`SignedInfo` describes how the content is signed, `KeyInfo` allows to identify
the signer and `Object` is the signed content[^1]. Since signing and verification
involve computing the hash (digest) and asymmetric cryptography, care need to be
taken to sign and verify exactly the same content. Those beasts are ruthless if
there's even a single byte changed. To get a canonical form of an XML content,
some uniform rules need to be applied so that two logically equal XML files look
exactly the same. Then the result can be either signed or verified.

Once you've got a glimpse of the whole process, you may want to (or someone
forces you to) do it programmatically. Now you might be tempted to call it a day
after finding `xmlC14NDocSave()` function in the [libxml2 API documentation](http://www.xmlsoft.org/docs.html). But
before you cross this bridge, you need to answer first what's your quest. And
believe it or not, this is not easy. The catch is that the [`xmlC14NDocSave()`](http://www.xmlsoft.org/html/libxml-c14n.html#xmlC14NDocSaveTo)
takes a set of nodes you want it to operate on and in fact it doesn't do all the
dirty job for you. You need to provide it a set of the _right_ nodes in the _right_
order. Here's the [XPath](http://www.w3schools.com/xpath/) incantation:

{% highlight terminal %}
descendant-or-self::* | descendant-or-self::text()[normalize-space(.)] |
.//attribute::* | .//namespace::* | .//comment()
{% endhighlight %}

There's some uncertainty whether to use `normalize-space(.)` or not and what the
relative order of attributes and namespaces should be. This unfortunately
depends on who you talk to and where you received signed XML from or where you
intend to send it. For example signed XML files in Adobe AIR packages require
space normalization while _xmlsec_ tool doesn't. This is mundane and brutal
reality of format incompatibilities. Beware.

## Getting hands dirty

Since I got your full and utter attention and you're so excited that you
probably dropped some of your late at-the-desk lunch onto your smart looking
office trousers (or onto your pants if you're home alone or in the United States
of America), let me show you some sample code. Full source code along with test
scripts is available [here](https://github.com/kkonopko/kriscience/tree/master/xmldsig-c14n). Here's the essence with mundane error checking
removed here for brevity:

{% highlight C++ %}
  // Now this is old granny's secret recipe for a delicious cheesecake.
  const xmlChar C14N_CONTENT[] =
    "descendant-or-self::* | descendant-or-self::text()[normalize-space(.)]"
    "| .//attribute::* | .//namespace::* | .//comment()";

  // Get some cheese... all sub-document content we need for canonicalisation.
  const auto sinfo =
    make_scoped (xmlXPathEvalExpression (C14N_CONTENT, ctx.get ()),
   xmlXPathFreeObject);

  // And finally... bake it!
  xmlC14NDocSave (doc.get (), sinfo->nodesetval, 0, NULL, 0, file_out, 0) < 0);
{% endhighlight %}

The full implementation is available [here](https://github.com/kkonopko/kriscience/blob/master/xmldsig-c14n/xmldsig-c14n.cpp). And here's how I compile it on my
Fedora 18 laptop:

{% highlight terminal %}
g++ -O3 -std=c++11 \
  $(pkg-config --cflags --libs libxml-2.0) \
  xmldsig-c14n.cpp \
  -o /tmp/c14n-exe
{% endhighlight %}

And here's a Bash one-liner that takes a signed canonicalised XML on its input
and verifies it (provided that you have one—see below what to do if you
don't). For convenience I split it into multiple lines but essentially it's a
single line command.

And here's a Bash one-liner that takes a signed canonicalised XML on its input
and verifies it (provided that you have one—see below what to do if you
don't). For convenience I split it into multiple lines but essentially it's a
single line command.

{% highlight bash %}
/tmp/c14n-exe "/default:Signature/default:SignedInfo" /tmp/sample.xml |
openssl dgst -sha1 -binary -verify <(
  printf -- \
    "-----BEGIN CERTIFICATE-----\n%s\n-----END CERTIFICATE-----\n" \
    "$(xmllint \
        --xpath "//*[local-name()=\"X509Certificate\"][1]/text()" \
        /tmp/sample.xml)" | \
  openssl x509 -pubkey -noout) \
  -signature <(
    xmllint \
      --xpath "//*[local-name()='SignatureValue'][1]/text()" \
      /tmp/sample.xml | \
    openssl base64 -d)
{% endhighlight %}

A more convenient script is [here](https://github.com/kkonopko/kriscience/blob/master/xmldsig-c14n/test-c14n.sh). If you don't have a XMLDSig file at hand, you
can generate one with the [script](https://github.com/kkonopko/kriscience/blob/master/xmldsig-c14n/generate-xmldsig.sh) I provided.

{% highlight terminal %}
./generate-xmldsig.sh /tmp/c14n-exe > /tmp/sample.xml
./test-c14n.sh /tmp/c14n-exe /tmp/sample.xml
{% endhighlight %}

That's it for now. Enjoy!

[^1]: Actually `Object` is digested and the digest is included in `SignedInfo`
which itself is digested and signed.

{% capture subject %}
{{ page.title }}
{% endcapture %}
{% include comments.html title=subject %}
