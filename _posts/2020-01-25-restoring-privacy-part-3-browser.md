---
layout: my-post
title: Restoring privacy (part 3) - Browser
date: 2020-01-25 11:33
---

No matter how hard I try to avoid it, a lot of content and services I use, are
available online.  Most of the time the most convenient way to access them is
with a web browser.  I'm on my crusade to limit my need for using a web browser,
but for the time being, small steps, I am where I am, and I need a "traditional"
web browser.

This is not going to be another review of web browsers, there are plenty of them
available and actually it keeps changing.  By the way, I don't quite trust
reviews as it's difficult to remain objective, yet reviews can be inspiring
(especially if a web browser receives similar reviews consistently).  Instead
this is my very subjective view and I share what lead me to my choice, which
might not be the final one yet.

## Sifting

As soon as I realised I don't want to use Chrome (after many years of using it),
I wasn't sure which browser to pick instead.  One natural choice for me which
would not turn everything upside down was Chromium.  I relied on
synchronisation of favourites and some plugins, but somehow didn't trust even
Chromium.

I think after going through the list at https://www.privacytools.io/browsers/, I
jumped onto Mozilla Firefox bandwagon.  I made all configuration t weaks to
remain super private.  But within a week or two, when I used it all the time for
work (Jira, code reviews etc.) and for personal use (separate profile), it
started to fall short for me.  Some web sites were slow to load, some didn't
load at all.  Some of these problems were related to my radical configuration
changes, for example disallowing some forms of identification.  But I was fed up
with it when I discovered that something with copying and pasting into Jira
tickets didn't work quite well.  I even raised a Jira ticket with the IT
department of the company I worked for.  Soon afterwards, I tried to reproduce
the problem with another browser (Chromium I think), where everything worked
fine.  Eventually, being very frustrated, I ditched Firefox completely.

I've tried other privacy oriented browsers based on Chromium but there was
always something that sooner or later didn't feel right to me.

Whatever the choice, there's also no guarantee something doesn't change about
the browser or the company behind it.  I recall some telemetric related changes
in Mozilla Firefox which raised many hackles.  Probably it's best to not become
dependent on a particular web browser and remain ready to switch.

## Brave

I'm not a big fan or whatever.  I tried it early on after ditching Chrome but
had problems with it, like some web sites didn't work for me.  But a couple
months ago I gave it another try and stuck with it for now.  It's been a while
since I've tried it for the first time, so I guess it was improved a lot.  Seems
to be fine.  As long as I'm not too paranoid about cookies and scripts settings,
all web sites seem to work well for me (used to have issues with Garmin Connect
specifically and its sign-in process).  I went through all the settings and
changed them to reasonable values.

Quite valuable for me at this point is also that it's got all Chrome plugins
available.  Something I'd like to change and become less dependent on, but
remember, small steps.  It works well with Enpass which I [use][enpass-post]
these days.  With some of the privacy oriented plugins (like 'PrivacyBadger',
'HTTPS Everywhere', 'Disconnect' etc.), at least I feel somewhat better.  I
guess there's no point in being paranoid, just remain sane.  I don't believe
it's trivial to remain private.  Heck, I think it's difficult, or at least it's
highly inconvenient.  Sad but true.

I'm also addicted to Amazon's 'Send To Kindle' plugin which I use for articles
which take me more than 5 minutes to read.  And yes, this could be a post on its
own, as I've tried other plugins for sending web content to my Kindle and none
of them worked as well as Amazon's.  It's a remarkable privacy concern,
especially being constantly signed in with Amazon on every web site I
potentially may want to send to my Kindle.  But that's the price for the
convenience I'm willing to pay at the moment, and I haven't found a better way
yet.  And no, I'm not willing to try other e-book readers, I'm not ready for
that yet.

I also like Brave's built-in "reader view", although while writing this post on
my Mac, and I can't find it in the Mac version of the Brave browser (I have a
'Reader View' plugin installed).  This is extremely useful to me on annoying
web sites which try to present too much garbage, or pester me with pop-up
windows (usually related to privacy and cookie settings).  I just want to read
the text!

## My other practices

I keep on using profiles for my personal browsing and work related browsing.  I
have a profile for the company I work for where I remain signed it everywhere
possible, I have no privacy concerns in this mode.  For my personal browsing I
try to use private tabs as much as possible.  Where it's more convenient and
somewhat trustworthy, I use the default (non-private) mode to remain signed in
(say, Mastodon or my own TinyTinyRSS instance).  For the rest of the browsing,
especially for accessing banking web sites, I try to use private mode.

## Distraction

I find web browsing distracting so I try to limit it.  When it's work related,
if I can find something in my local (offline) documentation, I try to use it.
Only when stuck, I try to search on the web.  Even in that case I try to use
text-oriented browsing (say, with 'eww' browser built into Emacs).  It's not the
best experience all the time, but I'm getting used to it and starting to
appreciate it.

I keep on improving my practices, and try to limit the distractions.  Or to put
it another way, remain focused and spend more time in deep work mode.  So for
instance I try to use text-mode browser, use 'org-jira' in Emacs instead of
going to Jira in the web browser, read Mastodon entries in Emacs 'mastodon' etc.
I try to write things up in text editor first and paste into the web browser
form.  I send longer articles to Kindle.  I try to not rely on history to make
it more tempting to use the web browser (do I really need to go to that web
site?).  In fact, the more I get used to these practices, actually it allows me
to get things done quicker.  I'm far from ditching a traditional web browser
completely, maybe never will, by my goal is to limit its use as much as
possible.

## Search engines

This will be short:  Startpage, Qwant, SearX, DuckDuckGo.  In this order of
preference.

It was a bit of a shock at first to switch away from Google for searching the
web and I kind of trusted the results from the other search engines less, but
these days actually it feels for me so much better and more accurate in fact.  I
can't remember when I used Google search engine the last time.

OK, I admit it, after trying really hard, for maps I came back to Google Maps in
private browsing mode.  I tried all sort of OpenStreetMap incarnations, even
preferred to stuck with Apple one (I still use it on my phone), but finally if I
really need to look something up in the web browser, it's Google Maps again.
Hopefully I'll be able to switch away from it at some point, again.  Maybe it's
one of those things I need to think about: do I really need it?  why do I need
it?

Ta!

## Updates

### 2020-03-18

At the moment I'm so distracted by using a mouse that I'm evaluating
`qutebrowser` which seems to be aboslutely brilliant.  When it comes
to reading up online content, I try `qutebrowser` first.  Only when
I'm unsure about something, I go back to `Brave`, which I still use
for banking (or anything requiring a login), have most of my bookmarks
saved (synced with `floccus`, although started adding some new in
`qutebrowser`).  I guess it's only a matter of time and re-evaluating
things like replacing `Enpass` with `pass` etc.

[enpass-post]: {% post_url 2020-01-04-restoring-privacy-part-2-passwords %}