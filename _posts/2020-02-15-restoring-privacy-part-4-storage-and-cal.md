---
layout: my-post
title: Restoring privacy (part 4) - Storage and Calendar
date: 2020-02-15 06:33
---

As much as cloud services make life easier, there are two types of them which
I've always felt bad about:  files and calendar.

I'm not including e-mail on that list.  I'm not ready, and maybe will never be,
to host my own e-mail server.  There are many reasons for it (maintenance,
handling power outages, reliability etc.) which all could be addressed, but in
the end e-mail communication requires at least two parties, where one of them
might not be as prudent about handling e-mails as I'd like to.  So inevitably, I
treat e-mail communication as not something I can have full control of.

This is different for _my_ files and calendar.  My main reason for considering
anything in the "cloud" to store my files and calendar is synchronisation across
multiple devices.  Further reasons to that are back-up and accessibility on
multiple devices.  The latter reason is somewhat questionable to me but the
reality is that people kind of expect me to be able to whip out my phone, check
my calendar and tell them when I'm available.

## Disqualifying cloud services

I had been a long time user of Dropbox.  This all started at the time when
privacy was not a concern and this kind of cloud services appeared to be a real
boon.  Possibly even at that time companies providing those services didn't even
had in mind exploring users privacy.  All my calendar was with Google and that
felt awesome as well.

When I begun restoring my privacy, files and calendar were my biggest
concern.  I quickly came across Nextcloud, but being not ready, I tried out some
publicly hosted instances.  It always bothered me though, especially that I had
no control over the features enabled on a particular instance, nor its version.

I also considered to keep on using public instances, or even return to Dropbox,
but with file encryption.  This is doable but it'd still nag me.

For the calendar, in the early days of my journey of restoring my privacy, I
evaluated Tutanota which at the time did not have calendar.  Apparently, at the
time of writing of this post, they seem to be doing quite well (I follow them).
They're still missing private file storage but as far as I can tell, this is
something they are planning to provide.  I'm no longer sure though whether I'd
give my data back to anyone again.  They are doing a great job to make it
convenient for people which for whatever reason are not in a position to handle
their data themselves, but to me it'd feel like taking a step back.

## Synology

I have a Synology server which attempts to make it super convenient to not only
own "cloud" services, but also physically own the host machine and data.  I do
appreciate this a lot.  I've tried its calendar, cloud storage (SynologyDrive),
notes (SynologyNote) and photo synchronisation (Moments).  It seemed easy to
install and configure but at times felt unreliable.  I got fed up once when all
my calendar events disappeared in the web version of the calendar.  After an
hour or so of investigating it, I found out that one of the events I created on
my iPhone made it really upset (it could not handle some Apple specific calendar
meta-data).  Also photo synchronisation worked worse and worse for me, and
eventually stopped working for reasons I've never figured out.

So regretfully I gave up on it and looked for an alternative.

## Nextcloud on FreeNAS

Eventually, I ended up with my own Nextcloud instance.  I was encouraged to try
it out as it was available as a plug-in on FreeNAS, which in the meantime I
played with and liked.  In fact, being more familiar with Nextcloud installation
now, I'm tempted to build it all myself from scratch as it'd allow me to be at
the latest version.  Yet I'm a bit reluctant to spend hours on finding out what
went wrong with data migration, and possibly I'd need to invest a bit of time to
build an iocage for it.

With Nextcloud I get both file storage and calendar, so it's got what I was
looking for.  Plus it supports WebDAV, and there's an iOS app and I can also
synchronise photos from my phone.  I use WebDAV for password synchronisation (at
the moment I'm using Enpass) and photos synchronisation with a Synology at home
(I can view them on TV over DLNA).  For the photos, I sold Nikon D80 few years
ago when I saw pictures I took (with some effort) compared to those which my
wife took with her iPhone (with no effort).  So being able to easily synchronise
photos is a great bonus.  Synchronising all of this on my laptops (Linux and
MacOS) is easy and well supported.

For all of this to make sense to me, I wanted to also have physical control over
the disk storage, so my FreeNAS/Nextcloud instance runs on my old desktop PC,
with disk encryption enabled.

My back-up strategy relies on files and calendars being distributed on multiple
devices.  So if any of them (including the server) fails, I still have a copy of
the data on other devices.  For the photos, which I do not synchronise all across
devices, they are synchronised to my Synology (mainly to be able to
view them on TV at home, as my FreeNAS server is located in my daily office),
which has an encrypted AWS back-up set up.

## Conclusion

As people rely more on digital solutions to organise their lives, I wish there
was a super convenient way of owning generated data and also the physical medium
where this data is stored.  Synology and similar NAS solutions make it somewhat
easier but even that is not something I'd recommend to everyone.  It should be
pretty much like a washing machine which is plugged in and that's it.  But even
with simple appliances, the appetite for IoT and seeing where this is all going
to, chances are high that the effort required to own data and protect privacy
will not become lower.

## Updates

### 2020-03-18

Since I screwed up with full-disk encryption in my Nextcloud/FreeNAS
installation (something failed, the disk stopped accepting the
password and remained locked after several unclean reboots), I'm back
on Synology safely kept at home: calendar, WebDAV and pretty much
everything except for email.  It's a pity it does not support full
desk encryption but for the time being it's a minor disadvantage.