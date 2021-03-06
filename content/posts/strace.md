---
title: "Strace"
date: 2013-06-02T13:03:00+01:00
draft: true
---

Sometimes when debugging on a unix-like OS, you need some extra help to figure
out what's really going on. Instead of treating certain processes as "black
boxes", the unix command "strace" allows you to peer into what's going on.

In this post we'll take an introductory look at Strace and how to use it.

### What is Strace?

The strace man page entry has the following to say about strace:
"strace is a useful diagnostic, instructional, and debugging tool.  System
administrators, diagnosticians and trouble-shooters will find it invaluable for
solving problems with  programs for which  the source is not readily available
since they do not need to be recompiled in order to trace them."

In short, strace allows you to attach yourself to a unix process and follow
along as the program makes various system calls ranging from I/O operations to
calls to the internet.

As the man page says "Each line in the trace contains the system call name,
followed by its arguments in parentheses and its return value"
