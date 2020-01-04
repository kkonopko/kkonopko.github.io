---
layout: my-post
title: Restoring privacy (part 2) - Passwords
date: 2020-01-04 07:01
---

It's overwhelming these days--everywhere I'm asked for a user name and a
_password_.  It'd be great if only there was a convenient and _widely adopted_
way of proving my identity, or more precisely, my rights to access something (if
I don't really want to expose my identity as such).  Meanwhile, I have to deal
with passwords somehow.

## Intro, skipped

There's plenty of resources (books, articles) which explain why having a good
password is important, and why not to reuse passwords.  I'm not going to
duplicate any of those.  The reality is that I need unique and strong passwords,
many of them, period.

There are also various ways of storing passwords, but what I'm after here is
picking a software password manager (vault).

## Enter password managers

It's a mess, like with pretty much everything, from picking a place to go for a
trip to choosing the "right" [email provider][email-provider].  There's so much
to choose from, which can easily cause a headache and eat up all of my (scarce)
spare time.  First thing then was for me to find out what I want, to somewhat
constrain the list of choices.

Here's my shopping list:
* offline
* portable (Linux, MacOS, iOS)
* custom fields
* something I could potentially sell to my family

Requiring it to work offline and yet remain portable means to me that the
password manager uses an encrypted store (file(s)) which I can synchronise
automatically using my own mechanism (something for another post).  I don't want
the password manager to mess with it.  This also means the same store can be
used across different platforms.

As much as I try to limit my use of the (smart) phone, the reality is that it's
too convenient to have my passwords available on it.  A truly paranoid
individuals would probably want to use a dedicated machine for example to access
their banking, and never really need passwords for anything important on their
phone.  Also it's healthier to stay away from phone as much as possible and plan
a task requiring a password to be completed for example on a laptop.  Compulsive
online shopping or logging into online banking to make a money transfer while in
a pub is no good.  But again, the reality is that it's easier to carry a phone
with me if I need to tweak some settings on my kid's tablet which may require a
password to the parent's account on that tablet.  Or something as tedious as
this.  And I want to use unique and strong passwords even (or especially for)
those mundane accounts.

I also try to encourage my family to manage their passwords as well.  In order
to support them, it's easier if they use the same solution.

## Choices

I'm pretty sure I haven't come across many passwords managers.  Maybe there's
still one that I missed and which would be the best choice for me.  I do not
feel like I'm done with it yet.

I also do not intend to enumerate all password managers I evaluated or read
about, except for only few of them.  This is also all very subjective.

For many years I had been using LastPass but became concerned about storing my
passwords in the "cloud", even if the promise is that they are encrypted and
safe.  I started to look for an alternative.

I loved the idea of [pass][pass-manager], the standard unix password manager.
Very neat, command-line based with plenty of UI clients and extensions.  The
problem I had with it at the time was that the iOS client required the passwords
to be synchronised with Git.  I didn't like the idea of either running my own Git
instance (which I might actually change my mind about at some point), nor the
idea of storing the files on any Git provider's instance, even as a private
repository.  Being lazy at the time I looked for an easier solution.

For some time I stuck with [KeePassXC][kee-pass-xc] which met all my criteria.
It worked well for me, even though the synchronisation with iOS client was
somewhat awkward (had to manually send the synced database file to the client
app).  Overall, it still felt like I was forcing myself to use it rather than
simply love it, which kept me open for alternatives.

I really liked [BitWarden][bitwarden], being open source and all.  The only
downside was that it used its own synchronisation mechanism.  Possibly I could
live with that by running my own instance but at the time I looked for an easier
solution.

## Enpass

Eventually I ended up with [Enpass][enpass].  Big warning:  it's closed source
which makes me really sad about it, but for the time being I decided to suck it up
as the features looked really plausible to me.  I don't even mind that the
iOS/macOS versions are paid to unlock handling more than 20 passwords (or
so)--at least a single license is enough to use it on multiple devices (there's
a limit) which is good enough to share it with my family.  It also does not
provide CLI, but again, for the time being the features it provides outweigh
what I'm missing from it.

What I like about it is that it handles TOTP tokens which means this factor is
synchronised across all my devices.  I just hate the idea of losing my phone
whilst being the only TOTP token provider.  Another nice thing about it is that
there's an option to also require unlock key (file) in addition to the master
password.  So even if someone captures my master password and seizes my password
database, they still need that unlock key which I may hold on a separate USB
dongle (and backed up if I lose the dongle).  On the phone the unlock key is
provided by scanning QR code.  Losing my phone, yet worse--someone gaining access
to it, is a complete different story.

I synchronise the database on my laptop through the file system and my own
Nextcloud instance while on other devices I use WebDAV protocol.  Works like a
charm.

## Conclusion

I still do not feel I have a perfect solution to managing passwords.  I'm open
for discovering something better.  I may likely reevaluate [pass][pass-manager],
and either overcome the problems I had with it, or possibly revisit my
requirements (simplify!).

[email-provider]: {% post_url 2019-12-29-restoring-privacy-part-1-email %}
[pass-manager]: https://www.passwordstore.org/
[kee-pass-xc]: https://keepassxc.org/
[bitwarden]: https://bitwarden.com/
[enpass]: https://www.enpass.io/
