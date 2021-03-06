    Title: Sandboxing vs. WINE
    Date: 2016-10-16T20:02:38
    Tags: DRAFT, security, lowlevel

There are two reasons I'm writing this post: one is that I've been
kind of meaning to write more about sandboxing (or, really, about
anything), and the other is that I was trying to run [Steam][] under
[WINE][] and got a half-broken GUI and a bunch of alerts that
`steamwebhelper.exe` had crashes.  So it's going to take a while to
get to the WINE.  With that out of the way:

[Steam]: https://en.wikipedia.org/wiki/Steam_(software)
[WINE]: https://www.winehq.org/

One of the central problems with [security sandboxing][] is that the
software ecosystem just wasn't designed for it.  It's bad enough if
you're trying to retrofit privilege separation onto a large
application that's been around for a while and wasn't remotely
designed for it, but there are also all the libraries you depend on —
including things you might think of as “part of the OS”.

[security sandboxing]: https://en.wikipedia.org/wiki/Sandbox_(computer_security)

By the textbook principles of modularity, interfaces should be
independent of their implementations, and your application shouldn't
be affected by how the lower layers of the stack interact with each
other.  Unfortunately, you've just revoked all filesystem access, or
installed a [system call filter][seccomp-bpf], or otherwise turned off
part of the OS API where people just assume they can do whatever.  And
now everything is broken and there's nothing you can do about it.

[seccomp-bpf]: https://www.kernel.org/doc/Documentation/prctl/seccomp_filter.txt

Or is there?  <!-- more --> The design pattern here is to apply enough
of the heavy-handed restrictions your OS gives you to stop what you
want to stop, and then somehow intercept the operations that would
have failed (but shouldn't have) to restore the things you didn't want
to stop by using some kind of interprocess communication to have an
unsandboxed (or less-sandboxed) broker do them on your behalf.

For example, on Linux you can (after entering an [unprivileged user
namespace][userns], if you're not root) take away all filesystem
access by [`chroot`][chroot]ing to a deleted directory, all network
access by unsharing the network namespace, and other very
coarse-grained things like that.  And you generally want to do *all*
of them.  (Trying to use syscall filtering as a sandbox by itself is
[probably a bad idea][sendmsg-oops], but using it as an extra layer of
security is [actually what seccomp-bpf was designed
for][seccomp-cr0].)

[sendmsg-oops]: https://bugzilla.mozilla.org/show_bug.cgi?id=1066750
[seccomp-cr0]: http://blog.cr0.org/2012/09/introducing-chromes-next-generation.html

But you can intercept raw system calls with [seccomp-bpf][] — the
filter can't do much on its own but it can raise `SIGSYS`, whereupon
the signal handler can do arbitrary things to “implement” the syscall,
then change the return value register in the signal context and
return.  Alternately, if you know that everything is going through
nice well-defined `libc` interfaces, you can use [ELF symbol
interposition][interpose].  And for brokering things like `open`, the
annoying but usable `SCM_RIGHTS` can pass file descriptors between
processes (carrying the privileges they had when opened).

[userns]: http://man7.org/linux/man-pages/man7/user_namespaces.7.html
[chroot]: https://en.wikipedia.org/wiki/Chroot
[interpose]: https://www.airs.com/blog/archives/307

Windows… I don't know very well, but there are [a bunch of things
Chromium pokes at][chrome-box], and clearly they're heavy-handed
enough that they need [a broker][chrome-broker].  

<!-- TODO: handles, maybe rearrange the Linux interception thing, then into binary patching and system call stubs -->

[chrome-box]: https://www.chromium.org/developers/design-documents/sandbox#TOC-Sandbox-restrictions
[chrome-broker]: https://www.chromium.org/developers/design-documents/sandbox#TOC-The-broker-process
