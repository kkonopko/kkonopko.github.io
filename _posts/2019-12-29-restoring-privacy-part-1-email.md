---
layout: my-post
title: Restoring privacy (part 1) - Email
date: 2019-12-29 16:26
---

It's been nagging me for a while, that something's possibly wrong with my online
presence and the services I had been using.  Particularly I had been a big
Google fan when it emerged.  It was really cool to have a GMail account at the
time, as an alternative to one of those providers in the dot-com boom era.  I
still think Google do a great job and their GMail service is really great and
_secure_ (yes, secure).  Also I can boast a _name.surname_ GMail account which is
actually not possible to get these days (I got it when GMail was getting
popular).  The problem with it is _privacy_.

Let's face it--Google is an advertising company.  They make money from selling
ads (actually auctioning them).  My data is their food.  I'm not even after the
fact that they are willing to give away my data to state agencies, as if that
mattered in my case, I'd have bigger problems.  I don't quite believe in
ultimately "safe" email, as whether it's PGP or any other means of protecting
it, someone will get to it if they are really after me (be it a flaw in the
software, service provider's mistake, my peer's mistake or my own mistake).  So
what really concerns me is blatant access to my email and information about me
which is being profiled and sold.

It's a fair deal: Google provides me a service "for free" (for no money) but I
pay with my privacy by letting them peek.  There's no big conspiracy here.
What's somewhat worrying though is that some users are not aware of the "price"
they are paying, or they do not care.  Not my problem.

Over the time, I've been scouring websites like [Restore
Privacy][restore-privacy] or [Privacy Tools][privacy-tools].  Also books by
Snowden, Mitnick and the like made it even more apparent to me that I needed a
switch.

## Restoring privacy

This post is the first one in a series I have in mind.  I'm not a privacy
expert, and yet where I have an opinion, I still try to remain pragmatic.  There
are a lot of opinions on which email provider to choose.  I've been through tens
of reviews.  The difficulty:  I had to make my _own_ choice after realising what
I really wanted or needed.

This is yet another opinion which may not be the right choice for everyone.

## Domain

One thing I'm quite certain about is that sticking with any email provider's
domain is not ideal.  My ambition is to close my GMail account at some point in
a (distant) future, but it'll take time and effort to update my account
information I'd given out.

What I've learned the hard way is that having my own domain is the first step.
I'm with [njal.la](https://njal.la/) and it works great for me.  They seem to
have all features, many of which I don't even know what they are for.  Recently
for example I found it quite useful to be able to set CAA record (after I
generated Let's Encrypt certificates for one of my websites) and realised
they have a nice dynamic DNS feature.  For paranoid they can be accessed on the
Tor network: [http://njalladnspotetti.onion](http://njalladnspotetti.onion) (I
use this ability for fun rather than for privacy).

So in the end, if I change my mind about the email provided (which has happened
to me a number of times), it's at least less painful to not having to update my
email address and only having to migrate existing messages.

## Email providers

There's a number of factors to consider here: whether the storage is encrypted,
the provider's jurisdiction, email clients (applications), webmail, custom
domain support, multiple address support etc.

I started off paranoid and tried ProtonMail.  Everything encrypted, based in
Switzerland etc.  What I disliked about it though is that I had to either use
their webmail (which somewhat annoyed me) or a bridge (daemon) to access my inbox
with IMAP/SMTP (if I chose to use a particular email client).

I also tried Disroot and eventually Tutanota.  With the latter one I still have a
spare account, just in case I turn to it again.  Tutanota takes things even
further (based in Germany): one can use either their webmail or their native
client as they want to guarantee nothing unencrypted is ever transferred.  They
also say that PGP is not the right choice and provide end-to-end encryption by
default (including meta-data).  The only problem with that is that my peers
would also need to be with Tutanota, or I should send them password-protected
messages.  And that would likely be only my closest circle as the majority of
the world which I need to contact, does not use Tutanota now nor in any near
future.

In the end I realised that email encryption is not the top priority for me as
the majority of my contacts would not use encryption.  If I really need extra
protection, PGP seems fair enough.  The default I wanted for my email was to
keep my email from preying eyes.

This is how I ended up with [Migadu][migadu].  I'm not going to review it here
as anyone can go and check their website.  I can summarise here that for mere
$4/month I can have:
* unlimited email domains
* unlimited addresses
* unlimited storage
* unlimited aliases
* proper IMAP/SMTP support
* _simplicity_

Sure, there are some conditions to use "unlimited" features reasonably but I
don't feel I'd ever need an "unreasonable" use of the service.  They do not
encrypt stored data but they are based in Switzerland and that's fair enough for
me.  First time around, the setup may look somewhat unusual but in the end it's
quite straightforward.  The only trap was to find out that with simplicity comes
responsibility and I had to set up catch-all rules myself (for example to have
addresses like `address+whatever@mydomain.tld` reach my inbox).  I had very
occasional glitches: once I could not sign in, reported it and apparently they
were doing some maintenance; the other time emails I knew were sent, arrived on
the next day.  But there's never a situation an email is lost (undelivered).  At
least that's the promise.

## Summary

Choosing an email provider is not an easy task.  There's an effort required to
find out what is really needed.  For me, there's always a balance I need to seek
but ultimately simplicity is a good indicator.  And it's fair enough for me if
my emails are simply not looked into.

In the next post I'm planning to share my experience of another fundamental
issue:  handling passwords.

[restore-privacy]: https://restoreprivacy.com/
[privacy-tools]: https://www.privacytools.io/
[migadu]: https://www.migadu.com/en/index.html
