---
layout: post
title: "HTML Redirects are Monadic"
date: 2013-05-03 07:37
comments: true
categories: 
---

In [Monadic i/o and UNIX shell programming][1], a correspondence
between a subset of the UNIX shell commands are shown to be isomorphic
to operations in Haskell.

While the conclusions may be somewhat academic--in that they only help
understand one in terms of the other.  The connection is nevertheless
worth further contemplation.

In what follows, I will show that many contemporary uses of HTML
redirects exibit an analogous behaviour to UNIX shell commands.
Furthermore, I will argue that due to this use, a distinction between
redirection and safe redirections must be made.  Where a safe
redirection is one that is not reflected visually in the browser's UI,
allowing possible improvements to user experience.

[1]: http://okmij.org/ftp/Computation/monadic-shell.html
