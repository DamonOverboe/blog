---
layout: post
title: "Ansible & Windows - Timeouts, Stack Overflows, Out of Memory"
date: 2014-08-30
categories: devops ci continuous-delivery ansible windows errors
---


I have been using Ansible & Windows to prepare my Jenkins clients. I
have used both the `win_msi` package from the mainline, as well as
Trond's `win_package` module which offers a lot of enhancements.

While testing out the base `win_msi`, I kept getting Out of Memory 
exceptions, Stack Overflows, HTTP 500s & timeouts. I found out about
Trond's enhancements to the `win_msi` module, as well as the addition of
the `win_package` module, and so I tried them as well, with the same
results. Sometimes, everything went fine; sometimes it did not.

Seeking help, first Trond teased me about it not being 1990 and told me 
to add more memory to the machines! :-D That was awesome, and made me
double-check that these weren't severely crippled, and they weren't.

What I was encountering turns out to be a bug in Windows *gasp* where it
limits the amount of memory given to WinRM to 150MB, despite whatever
the environment reports is allocated. *(Mine was on Server 2012, and was
defaulted to 1GB)*

See [this thread][thread] for more information; it covers a few topics
but about half-way down you'll see them discuss it. If that is the
error you're encountering, it's corrected by [KB 2842230][], which at
the time of this writing, is still undergoing testing and is not in the
Windows Update catalog yet; you have to manually download it and apply
it.


[thread]: https://github.com/ansible/ansible/pull/8345#issuecomment-52074837
[KB 2842230]: http://support.microsoft.com/kb/2842230
