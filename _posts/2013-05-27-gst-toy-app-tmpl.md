---
layout: my-post
title: GStreamer toy application template
date: 2013-05-27 21:28
---

This time I'd like to share some piece of code I find very useful when
experimenting with [GStreamer](http://gstreamer.freedesktop.org/). This includes
writing some small tests to understand some features and all sort of debugging
of specific cases after distilling them from larger applications.

## Rationale

The thing is that I don't want to start over and over again with the common
stuff whenever I want to quickly hack a GStreamer application. This is mostly
about writing the `main()` function and some bits and pieces:

* parse command line options
* initialise GStreamer
* create the pipeline
* optionally add a probe
* optionally send EOS

At the same time I want to keep it simple and not turn it into some versatile
utility which would obfuscate the main idea. Plain simple. I know there's
[gst-template](http://cgit.freedesktop.org/gstreamer/gst-template) project but
as I understand it's more GStreamer plugin oriented and helps with some
GLib/GObject boilerplate.

## The template

The goal for this "template" is to be reusable but mostly for some throw-away
prototypes or tests. But still I wanted to keep the quality as high as possible,
include proper error checking etc. as in practice a lot of prototype code ends
up in production.

The cool thing about it is that it reads the pipeline description from the
command line which is very useful when experimenting/debugging. It also offers
sending EOS event upon SIGINT (-e), adding probe on a pad (-p) and verbose
output (-v) if e.g. one wants see the output of a identity element. There are
few gaps to fill in, all marked with `TODO`, e.g. the type of probe and probe
return value. As I said I didn't want to over-complicate this so I left it to
edit in the code.

## Building and running

The source code is available
[here](https://github.com/kkonopko/kriscience/blob/master/gstreamer-app-template/gst-test-app.c). This
time I'm using [uninstalled](http://cgit.freedesktop.org/gstreamer/gstreamer/tree/scripts/create-uninstalled-setup.sh) version of GStreamer's master branch. On my Debian
laptop I build it as follows:

{% highlight terminal %}
libtool --mode=link \
  gcc -O2 \
  $(pkg-config --cflags --libs gstreamer-1.0) \
  gst-test-app.c \
  -o /tmp/test
{% endhighlight %}

And a couple of tests:

{% highlight terminal %}
[kris@lenovo-x1 master]$ /tmp/test -e fakesrc ! fakesink
** Message: Running...
^C** Message: Handling interrupt:  forcing EOS on the pipeline

** Message: End of stream
** Message: Returned, stopping playback
{% endhighlight %}

{% highlight terminal %}
[kris@lenovo-x1 master]$ /tmp/test -e -p "sink:sink" videotestsrc num-buffers=5 ! fakesink name=sink
** Message: Successfully installed probe on 'sink:sink'
** Message: Running...
Got event: stream-start

** (lt-test:27135): WARNING **: TODO: Implement me!!!
Got event: caps

** (lt-test:27135): WARNING **: TODO: Implement me!!!
Got event: segment

** (lt-test:27135): WARNING **: TODO: Implement me!!!
Got event: eos

** (lt-test:27135): WARNING **: TODO: Implement me!!!
** Message: End of stream
** Message: Returned, stopping playback
{% endhighlight %}

_Voilà_! Enjoy!
