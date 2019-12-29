---
layout: my-post
title: Handling POSIX signals
date: 2013-02-02 16:50
---

When programming for _*nix_ systems, like it or not, sooner or later your
application will be bothered with [POSIX
signals](http://man7.org/linux/man-pages/man7/signal.7.html). And even if
signals are considered to be a [broken design](https://lwn.net/Articles/414618),
you have to live with them, hammered with some trepidation. And even if you've
tinkered with it and managed to install some handlers, you can't really do much
in a signal handler context. You can merely set some flag and get out of there
if you don't want to get into trouble. And if you have to whack some threads in,
you feel completely screwed as there's no certainty whatsoever of which thread
would receive signals. A bit intimidating, isn't it?


There's an attempt to remedy this creepy situation with
[sigwait](http://man7.org/linux/man-pages/man3/sigwait.3.html)(3) and there's a
nice [article](http://www.linuxjournal.com/article/2121) about it. So fear no
more, you're not stranded as there's even a better solution:
[signalfd](http://man7.org/linux/man-pages/man2/signalfd.2.html)(2). It gives
you a file descriptor so you can choose how to handle signals—be it blocking
[read](http://man7.org/linux/man-pages/man2/read.2.html)(2) in a separate
thread, some sort of [poll](http://man7.org/linux/man-pages/man2/poll.2.html)(2)
which nicely integrates with polling loops
([`GMainLoop`](http://developer.gnome.org/glib/stable/glib-The-Main-Event-Loop.html0),
[`DBusWatch`](http://dbus.freedesktop.org/doc/api/html/structDBusWatch.html)
etc.)  or anything else you'd like to do with a file descriptor.

## The problem

Before I show an example of one approach to signal handling, I'd like to
elaborate a bit more on problems you may encounter with "traditional" signal
handling. It'll be easier for me to use a multi-threaded application example
although single-threaded applications are also negatively affected but in a more
subtle manner. Feels a bit contrived but I've seen it in the field.


Let's suppose someone wants to use a thread synchronization mechanism (e. g. a
mutex) in a signal handler. Signals are considered to be software interruptions
and their handlers are executed uninterrupted. If someone attempts to protect
some shared (global) data with thread-based locks, it will apparently lead to a
deadlock:

{% highlight C++ %}
int globalData;
std::mutex globalMutex;

void foo()
{
  // ... some stuff

  {
    std::lock_guard<std::mutex> lock(globalMutex);
    globalData = 1; // deal with the global data
  }

  // ... some other stuff
}

void handler()
{
  std::lock_guard<std::mutex> lock(globalMutex);
  if (globalData == 0) // deal with the global data
  {
    // ... something miserable
  } else {
    // ... something funny
  }
}
{% endhighlight %}

It's conceivable that `foo()` can grab the mutex right before the process receives
a signal. The `handler()` function is called as the signal handler. It tries to
get the mutex without a success and it ends up with a deadlock since it is not
scheduled out and doesn't let `foo()` release the mutex. Completely botched.

On the other hand, we cannot leave global data unprotected since there's no
guarantee that the data can be accessed atomically. We could use atomics here
but some architectures still use
[gUSA](http://lkml.indiana.edu/hypermail/linux/kernel/0205.2/1074.html) to
provide atomicity which might complicate things in the context of a signal
handler and in general the resource you want to protect might not be as simple
as a single atomic structure.

## The solution

I chose to civilize signal handling by making it thread-friendly (I can use
mutexes and what not). The signal handler runs in a dedicated thread but there
are other ways of doing it as the signal delivery is serialized by
[signalfd](http://man7.org/linux/man-pages/man2/signalfd.2.html)(2). You may
even use it in a single-threaded application where you will probably have some
sort of a main loop and check for any signals delivered to the application
process at your leisure.


The example application is available
[here](https://github.com/kkonopko/kriscience/blob/master/handling-signals/signal-handler.cpp). Below
I present only the main idea.

{% highlight C++ %}
// protect some shared resource, let it be the standard output
std::mutex g_mutex;

void signal_handler(const int fd) {
  struct ::signalfd_siginfo si;
  
  while (const int bytes = ::read(fd, &si, sizeof(si))) {
    std::lock_guard<std::mutex> lock(g_mutex);
    
    switch (si.ssi_signo) {
    // Handle signals here
    }
  }
}

int main() {
  ::sigset_t mask;
  ::sigfillset(&mask);
  // block all signals
  ::pthread_sigmask(SIG_BLOCK, &mask, NULL);

  // unblock all signals but deliver through a file descriptor
  const int fd = ::signalfd(-1, &mask, 0);
  
  std::thread t(signal_handler, fd);
  
  // do some other stuff
}
{% endhighlight %}

First it blocks all signals and then unblocks them in the
[signalfd](http://man7.org/linux/man-pages/man2/signalfd.2.html)(2) call so they
all are delivered through a file descriptor. The signals do not affect the
application (except for *SIGKILL* and *SIGSTOP*) until the application reads from
the descriptor and decides what to do about them. Nice and elegant.

And here's an example session on a x86 Linux PC with GCC 4.7.2.

{% highlight terminal %}
$ g++ -O3 -std=c++11 signal-handler.cpp -pthread -o /tmp/test
$ !$ &
[1] 4135
$ kill -SIGUSR1 %
Received signal 10
$ kill -SIGUSR2 %
Received signal 12
$ kill -SIGTERM %
Quitting application...
{% endhighlight %}

Bear in mind that any approach to signal handling assumes you have control over
the main thread of the application. If you are writing a library that is called
from the main application that you can't control, then you're thrown back at its
way of signal handling and other libraries it uses. This is a difficult
situation and frankly there's no robust solution to it. Every new thread
inherits signal mask from its parent thread, so when it's started it potentially
has all signals unblocked until it blocks them with
[pthread_sigmask](http://man7.org/linux/man-pages/man3/pthread_sigmask.3.html)(3)
call. Until that point new threads may still receive unsolicited signals.


Having the caveat above in mind I wish you best luck with signal handling. It
doesn't have to be difficult. Just try to push traditional way of doing it
toward oblivion and embrace
[signalfd](http://man7.org/linux/man-pages/man2/signalfd.2.html)(2) as soon as
you can.

## Final notes

Quite interesting analysis of the problem:
[http://www.macieira.org/blog/2012/07/forkfd-part-1-launching-processes-on-unix](http://www.macieira.org/blog/2012/07/forkfd-part-1-launching-processes-on-unix)

An interesting option if you happen to use GLib:
[`g_unix_signal_add()`](http://developer.gnome.org/glib/stable/glib-UNIX-specific-utilities-and-integration.html#g-unix-signal-add)
