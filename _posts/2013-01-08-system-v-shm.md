---
layout: post
title: System V shared memory fun
date: 2013-01-08 22:58
---

One of the reasons I mention it here is a nostalgia as my first exposure to
parallel programming in the academia was with the use of processes and System
V. If you're too cool for old skool, better start with threads instead. The very
first *nix system I ever used was FreeBSD. The very first *nix system I happened
to use in my professional career was Solaris. And I still use hjkl to move
around in vi since once I was given access to a Solaris server terminal with no
arrow keys on the keyboard and vi being the only editor available on the system
(visiting a large IT company in Asia as a support engineer). For these and other
reason I'm probably becoming an old grumpy man.

## Why, oh why?

Let me begin with a disclaimer that if you ever have a chance to deal with
shared memory, you're better off with [POSIX shared memory
API](http://man7.org/linux/man-pages/man7/shm_overview.7.html) or simply use
files and [mmap](http://man7.org/linux/man-pages/man3/mmap64.3.html)(2). You
still need some means of synchronisation though, like atomic operations and good
spinning strategies. System V shared memory
([shmget](http://man7.org/linux/man-pages/man2/shmget.2.html)(2),
[shmop](http://man7.org/linux/man-pages/man2/shmat.2.html)(2), etc.) on the
other hand is an old skool toolkit.

Nostalgia is of course not a good enough reason to mention System V shared
memory. In fact I'm not that old and my career is less than ten years but I've
come across a few projects where it was used. Although I was lucky to get a good
start from my academia on this subject, I had to learn more about it the hard
way. Maintaining and improving code around it has taught me some
lessons. Whatever one may think of this API, it's still quite popular and more
available in a variety of systems than the better POSIX alternative.

## A smidgen of an introduction

Good old books about the theory of parallel programming talk about semaphores,
processes and use a historical *V* and *P* notation for operations on
semaphores. See an overview
[here](https://en.wikipedia.org/wiki/Semaphore_(programming)). And if you're
stuck to a [fork](http://man7.org/linux/man-pages/man3/mmap64.3.html)(3) and
[execv](http://man7.org/linux/man-pages/man3/exec.3.html)(3) pair, better forget
about the latter one here. It's mind bending sometimes when there's a lot of
forking going on but no exec-ing. With some strategy applied we should be fine
though.

## Example

Let's have a look at a problem where there's a one server and multiple
clients. All parties need to send (share) data somehow. They also need some
exclusion mechanism to use a shared resource which in this case is a dazzling
computing power (the server). A scenario where a client wants to use the server
would be as in the following drama:

```
Scene:
  somewhere in an inter-process jungle

Characters:
  Master of puppets
  Server
  a number of clients

Appliances:
  3 semaphores: SemClient, SemServer, SemResultReady

Act I (prologue):
  Master of puppets enters the stage and initializes 1 semaphore to "up"
  (or "unlocked" if you're too cool for old skool)
  
  Master of puppets:
    V (SemServer)

  Two other semaphores are left "down" (or "locked" if you're too cool...)
  
  Master of puppets:
    Fork (Server)       [Let it be a server]
    Fork (Client)       [Let it be a client]
    Fork (Client)       [Let it be a client]
    ...

Act II:
  The server enters timidly the scene

  Server:
    P (SemClient)       [Is there anybody here?]
    
  A client enters the scene with a stately procession
  
  Client:
    P (SemServer)       [I'd like to speak to the server in private]
    
  The client gets private access to the server
  
  Client:
    (produces data in the shared memory area)
    V (SemClient)       [Hey server, here's some data I'd like you to munge]
    P (SemResultReady)  [Let me know when you're done]
    
  Server:
    (processes data in the shared memory area)
    V (SemResultReady)  [Mmm.. yummy data, here's the result]
    
  Client:
    V (SemServer)       [Thanks. Bye!]
    
Act III (epilogue):

  Master of puppets decides to cease everyone.
  
  Master of puppets:
    JoinAll ()
    
  Master of puppets leaves the scene
```

This boils down to the following lines:

```
Client:
  // get exclusive access to the server (and the shared memory area)
  P (SemServer)
  (generate some data in the shared memory area)
  // notify the server about the data to process
  V (SemClient)
    
  // wait for server
  P (SemResultReady)
  (consume the result)
  
  // release server access
  V (SemServer)
    
Server:
  // wait for a client
  P (SemClient)
  (process the data)
  // notify the client
  V (SemResultReady)
```

## Implementation

I try to simplify things by isolating the mind bending bits and writing things
in such a way that I can focus on the gist of the problem rather than nasty
details. I also try to use concepts from the problem domain. That's the strategy
I mentioned before to cope with the complexity.

Sources are available
[here](https://github.com/kkonopko/kriscience/tree/master/sysv-ipc-example). The
way I compile the sources and run the program is as follows (x86 Linux/Fedora
17, GCC 4.7.2):

{% highlight terminal %}
##
# Version without debugging messages
#
$ g++ -Wall -DNDEBUG -O3 -std=c++11 \
sysv-ipc-example/*.cpp -I./sysv-ipc-example -o /tmp/ipc-test

$ !$
/tmp/ipc-test
Client 4315
 result   : 863000
 expected : 863000 [OK]
Client 4316
 result   : 863200
 expected : 863200 [OK]
Client 4317
 result   : 863400
 expected : 863400 [OK]
Client 4318
 result   : 863600
 expected : 863600 [OK]
Client 4319
 result   : 863800
 expected : 863800 [OK]

##
# Version with some drama to follow (no -DNDEBUG)
#
$ g++ -Wall -O3 -std=c++11 \
sysv-ipc-example/*.cpp -I./sysv-ipc-example -o /tmp/ipc-test

$ !$
Start main 4345...
Starting server 4346...
Starting client 4347[Server] P (SemClient)

Starting client 4348[Client 4347] P (SemServer)

[Client 4347] V (SemClient)
[Client 4347] P (SemResultReady)
[Server] V (SemResultReady)
[Server] P (SemClient)

... (lots of drama)

Client 4351
 result   : 870200
 expected : 870200 [OK]
[Client 4351] V (SemConsole)
Terminating client 4351
Finished waiting for 4351
Terminating main...
~IpcManager() : freeing resources in 4345
{% endhighlight %}

Let's have a look at some code excerpts in
[client-server-example.cpp](https://github.com/kkonopko/kriscience/blob/master/sysv-ipc-example/client-server-example.cpp). `Client()`
and `Server()` almost replicate what I sketched about their roles above. To make
the program finite there's some expected (constant) number of clients and
transactions to occur but in an undetermined order. The data "protocol" between
the server and the clients is established by the Packet structure. Note that
there's also additional semaphore SemConsole to avoid interleaving log messages
from processes. The essence of the client implementation looks as follows:

{% highlight C++ %}
void
Client ()
{
  /* ... */
  
  int sum = 0;
  Packet *const pData = static_cast (GetShm ());
  
  for (unsigned i = 0; i < NO_OF_PACKETS_PER_CLIENT; ++i) {
    // wait for server access
    CLIENT (P (SemServer))
    // generate data:
    // easy to predict the result but still individual for every client
    std::fill_n (pData->numbers, NO_OF_ITEMS_IN_PACKET, pid);
    // notify the server
    CLIENT (V (SemClient))
    
    // wait for the server and get the result
    CLIENT (P (SemResultReady))
    sum += pData->result;
    
    // free server access
    CLIENT (V (SemServer))
  }
  
  /* ... */
}
{% endhighlight %}

I use `CLIENT` and `SERVER` macros to add some debugging messages so they form a
nice story on the console.

Now some juicy details of the API usage in
[`IpcManager.cpp`](https://github.com/kkonopko/kriscience/blob/master/sysv-ipc-example/IpcManager.cpp). For
example the *V* operation looks like this:

{% highlight C++ %}
void
IpcManager::V (const unsigned short sem) const
{
  ::sembuf sb = { sem, 1, 0 };
  const int status = ::semop (m_semId, &sb, 1);
  CHECK (0 == status);
}
{% endhighlight %}

That's it. I invite you to read
[semop](http://man7.org/linux/man-pages/man2/semop.2.html)(2) rather than
repeating what's already explained there. And the last but not the least, how we
initialize all that stuff:

{% highlight C++ %}
// only the first instance of the manager (creator) is responsible
// for resources
IpcManager::IpcManager (const char *key, const size_t memSize, const int semNum)
  : m_bCreator (true), m_data (nullptr)
{
  assert (key && key[0]);
  m_key = ::ftok (key, key[0]);
  CHECK (-1 != m_key);
  
  m_memId = ::shmget (m_key, memSize, IPC_CREAT | 0600);
  CHECK (-1 != m_memId);
  
  m_semId = ::semget (m_key, semNum, IPC_CREAT | 0600);
  CHECK (-1 != m_semId);
  
  for (int i = 0; i < semNum; ++i) {
    const int status = ::semctl (m_semId, i, SETVAL, nullptr);
    CHECK (-1 != status);
  }
}
{% endhighlight %}

Note that the key passed to
[ftok](http://man7.org/linux/man-pages/man3/ftok.3.html)(3) is literary a key
that identifies a shared resource so that processes know which one to connect to
in order to communicate (it's a bit like a token for a conference call). Note
that the key must be actually a path to an existing file system object.

Bear in mind that all processes are forked from the very first one. This means
that after forking all of them have their memory in the exactly same state as
their parent. This also means that all variables have exactly the same values
(which could be problematic if you fork from a multi-threaded process and have
some mutexes locked and only one thread left in the child process that waits for
some of the locked mutexes). As System V API resources have to be allocated and
deallocated somewhere (see the Gotchas section), we declare only one process to
be responsible for both operations. It doesn't have to be like this, it just
have to be done once. The IpcManager object is initialized in the first process
so it becomes a "creator". The same object manages forking new processes and
ensures they won't attempt to de-initialize the resources.

## Gotchas

### Managing resources

To me the most difficult aspect of System V API is ensuring reliable shared
memory deallocation. Bear in mind that if you don't deallocate it, _the system_
(not just a single process) will be leaking and sooner or later the host will
not be able to operate properly if there are new processes requesting System V
memory. A modern system usually has only few megabytes of System V memory
available so it's not something you can
splurge. [ipcs](http://man7.org/linux/man-pages/man1/ipcs.1.html)(1) and
[ipcrm](http://man7.org/linux/man-pages/man1/ipcrm.1.html)(1) are your friends
here.

What makes the deallocation actually tricky is deciding who should do it. If the
process that was supposed to deal with it crashed, then we've got a
problem. Different strategies can be applied depending on the situation but they
tend to be complicated. In this regards aforementioned [POSIX shared memory
API](http://man7.org/linux/man-pages/man7/shm_overview.7.html) seems to be much
better.

### Memory segment initialization

Although the [shmget](http://man7.org/linux/man-pages/man2/shmget.2.html)(2)
documentation promises:

> When a new shared memory segment is created, its contents are initialized to
> zero values [...]

that's certainly not true on all systems. I spent days chasing a problem on a
server farm consisting of tens of machines of different makes, hardware,
systems, versions etc. After contriving literary a hunt, it appeared that the
problem was only on particular machines and was diagnosed to be non-conforming
implementations not initializing shared segment.

Now I'm a bit sceptical about such promises if the software is intended to run
on undefined hardware and system. Unless it's a promise made by a portable and
well maintained library, I prefer to ensure that a resource I'm given is
initialized properly.

## Final notes

A nice reference for all this stuff is here:
[http://www.tldp.org/LDP/lpg/node21.html](http://www.tldp.org/LDP/lpg/node21.html)

An example of System V constraints: [https://github.com/android/platform_bionic/blob/master/libc/docs/SYSV-IPC.TXT](https://github.com/android/platform_bionic/blob/master/libc/docs/SYSV-IPC.TXT)

{% capture subject %}
{{ page.title }}
{% endcapture %}
{% include comments.html title=subject %}
