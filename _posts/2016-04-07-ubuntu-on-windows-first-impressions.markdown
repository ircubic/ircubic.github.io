---
layout: post
title:  "Ubuntu on Windows: First impressions"
date:   2016-04-07 10:00:00
categories: blog
---
Ubuntu on Windows was just released to the (Insider) masses yesterday, and
I've had a bit of time to play with it. It works surprisingly well, but the
places where it doesn't work stick out like a sore thumb, and hamper it from
truly being a replacement for a linux-based VM.

![apt-get on windows?!](/images/uow-terminal.png)

Some of the issues stem from the choice of implementing it as a subsystem and
will probably never properly be addressed, but others seem like issues that
could be fixed within the limitations of the subsystem.

This is the short list of basic limitations I've found so far in the Ubuntu on
Windows system:

 - Limited to 64-bit packages, 32-bit binaries don't run.
 - Can't run Windows binaries. (just like real linux!)
 - Can't run any services, because upstart is unavailable.
 - No actual linux kernel so things that interact with hardware either plain
   don't work, or work improperly.
 - No windowing (a consequence of the previous point), so no graphical apps.
 - Terminal emulation is incomplete. (This issue, to my knowledge, is being worked on.)
 - POSIX-y filesystem operations on the Windows drives (`/mnt/c` et al.) don't work,
   either succeeding and doing nothing or straight up crashing. (changing permissions, making symlinks)

Two of these issues come together to cause the biggest hangup I have with this
solution: you can't run any servers. You can't start any services with
upstart, and even if you start them the old-fashioned way with
`/etc/init.d/nginx start`, or similar, the inability to talk to the network
hardware means you can't bind to ports, so the server just fails to start.

You also can't run the server software you may have installed under Windows,
but that's not too surprising and is not likely to change. One downside of
this "linux as a subsystem" thing is that linux only understands linux-based
ELF-binaries and won't be able to run windows-based PE-binaries.

This means that while you can use the environment for scripting and file
manipulation, if you're trying to develop server-based apps on windows you
can't actually run them from the linux subsystem. This is a big deal-breaker
for me, and one that will have to be addressed for this solution to be more
than just an alternate way of manipulating files.

While playing with it I also came across some other limitations or quirks,
mostly related to secondary package managers, interacting with the file system
and terminal emulation:

When accessing the Windows drives, the POSIX permissions are essentially just
for show. They can't be altered, everything is 777 and owned by root:root, but
you're still limited by your current Windows user's permissions. This means
that it will appear that you can access other user's files, but if you try you
get an error. You CAN add linux users (say, if you want to use the same home
directory as your Windows user, instead of `/root`, which is the default), but
due to the permissions issue there is little point unless you enjoy sudo-ing
all the time.

In addition, it's impossible to create symlinks on the Windows drives (which
precludes running some software that rely on them from Windows drives), and
the system can get very confused with Windows' "magic" folders, which show up
like this:

![How helpful](/images/uow-magic.png)

As for the terminal emulation, it works kind of like when you ssh into a
server with the wrong terminal settings. You can't scroll properly in less,
you can scroll down just fine, but scrolling up only updates the top line. If
you've resized your terminal, most apps still think your terminal is default
sized, which is especially annoying with apps like vim. `top` and `htop` plain
don't work, and in one instance messed up the terminal to the point where I
had to reset it.

This, as mentioned before, is a point where Microsoft is actively going to
improve the solution in the near future, from what I've read, and it works
just well enough to be able to use.

It also seems that both `npm` and `gem` run into problems installing some
packages. Mostly this seems to relate to packages with native modules or
symlinks, but `npm` also fails seemingly randomly. I managed to install
Express.js by simply running the command repeatedly until it succeeded.

All in all, it's a promising start, with some very obvious sore points that
have to be addressed if the solution is going to be anything more than just a
toy and a neat way to get access to basic linux tools like `grep` without
having to install cygwin/msys.
