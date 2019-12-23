---
layout: post
title: Hook up a debugger to a stubborn application
date: 2012-11-25 18:11
---

Usually debugging an application is straight forward: fire it up with a
debugger. Even if the application requires some context (environment variables,

specific directory, arguments etc.), it's not that difficult as either it's
scriptable or the IDE will offer a plethora of options.

Now what to do if it comes to debug an application with a tortuous launch
procedure, e. g. through a number of scripts and/or some helper processes?
Certainly one may eventually find a way to force IDE/debugger to handle it but
as a lazy programmer you don't want to go through all of this just for the sake
of dirty debugging, do you?

A dirty solution for a dirty debugging is to start the application with whatever
requires it to start, make it wait for us and our dirty debugger to attach and
then continue under the debugger control. _This approach assumes that we can edit
our application source code and rebuild it more easily than finding a way to
start it with the debugger_. This includes a situation when we just want to debug
a code in a tiny library that is loaded by some unpleasant application.

Good. Now let's get down to the dirty details.

{% highlight C++ %}
for ( bool bDbg = true; bDbg; ) {}
{% endhighlight %}

I know, it looks utterly silly but it expresses the concept. It will loop until
someone tells it to stop. The point is that someone is a "third person", like in
a crime story. We'll see later who it is and how they tell the loop to stop.

An improved version of the silly loop is a daft loop:

{% highlight C++ %}
for ( bool bDbg = getenv("KRIS_DBG"); bDbg; ) {}
{% endhighlight %}

This one at least exposes its folly only when the environment variable `KRIS_DBG`
is visible to the application process. Put the loop somewhere in your
application/library code where you want to start your debugging session. This
will work even for an optimized code as there's not much the compiler can do
about the code around bDbg variable. Good for us.

Example:

{% highlight C++ %}
#include <cstdlib>
#include <iostream>

int main() {
    for ( bool bDbg = std::getenv("KRIS_DBG"); bDbg; ) {}

    std::cout << "There you go" << std::endl;
    return EXIT_SUCCESS;
}
{% endhighlight %}

To make this example more interesting, let's compile it with optimizations so
referring directly to `bDbg` variable is impossible (the compiler in this mode
doesn't "allocate" it as an entity in the generated code). This might be
reasonable if we want to debug an optimized program when we investigate program
crash which doesn't occur when compiled with debug information (due to different
process memory layout imposed). So here we go with the command line (Linux, x86
PC):

{% highlight terminal %}
$ g++ -O3 -o test test.cpp
$ KRIS_DBG=1 ./test &
[1] 1258

$ gdb -p 1258
... some GDB introductory stuff ...
{% endhighlight %}

OK, I know it's lame but I'm Intel syntax (l)user. That's what I've been taught
in the nursery school.

{% highlight terminal %}
(gdb) set disassembly-flavor intel
{% endhighlight %}

Now let's see how our silly loop looks in the disassembly.

{% highlight terminal %}
(gdb) x/2i $pc
0x8048815 <main+245>: lea    esi,[esi+0x0]
0x8048818 <main+248>: jmp    0x8048815 <main+245>
{% endhighlight %}

Rather boring. Execute two instructions in a busy loop. Let's have a look at
first 20 lines of the `main()` function:

{% highlight terminal %}
(gdb) x/20i main
0x8048720 <main>:    lea    ecx,[esp+0x4]
0x8048724 <main+4>:  and    esp,0xfffffff0
0x8048727 <main+7>:  push   DWORD PTR [ecx-0x4]
0x804872a <main+10>: push   ebp
0x804872b <main+11>: mov    ebp,esp
0x804872d <main+13>: push   edi
0x804872e <main+14>: push   esi
0x804872f <main+15>: push   ebx
0x8048730 <main+16>: push   ecx
0x8048731 <main+17>: sub    esp,0x118
0x8048737 <main+23>: mov    DWORD PTR [esp],0x80488e4
0x804873e <main+30>: call   0x80485a8 <getenv@plt>
0x8048743 <main+35>: test   eax,eax
0x8048745 <main+37>: jne    0x8048815 <main+245>
0x804874b <main+43>: mov    DWORD PTR [esp+0x8],0x4
0x8048753 <main+51>: mov    DWORD PTR [esp+0x4],0x80488ec
0x804875b <main+59>: mov    DWORD PTR [esp],0x8049b20
0x8048762 <main+66>: call   0x8048618 <_ZSt16__ostream_insertIcSt11char_traitsIcEERSt13basic_ostreamIT_T0_ES6_PKS3_i@plt>
0x8048767 <main+71>: mov    eax,ds:0x8049b20
0x804876c <main+76>: mov    eax,DWORD PTR [eax-0xc]
{% endhighlight %}

Now all we want to do is to force program to continue execution from the
instruction that succeeds the jump instruction `<main+37>` that led us to the busy
loop. Let's continue here: `<main+43>`. Shall we?

{% highlight terminal %}
(gdb) set $pc=0x804874b
{% endhighlight %}

Now we should be able to set breakpoints, watchpoints etc. and start
debugging. In this simple example I just let the program to continue until it
exits.

{% highlight terminal %}
(gdb) cont
Continuing.
There you go

Program exited normally.
(gdb)
{% endhighlight %}

If the program is compiled with debug information so that referring to bDbg
variable is possible, all the chaff above can be reduced to

{% highlight terminal %}
(gdb) set bDbg=false
{% endhighlight %}

Happy debugging!
