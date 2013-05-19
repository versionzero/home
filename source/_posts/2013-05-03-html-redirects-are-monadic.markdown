---
layout: post
title: "HTTP Redirects are Monadic"
date: 2013-05-03 07:37
comments: true
categories: 
---

In [Monadic i/o and UNIX shell programming][1], a correspondence
between a subset of the UNIX shell commands are shown to be isomorphic
to operations in Haskell.

While the conclusions may be somewhat academic--in that they only help
understand one in terms of the other.  The connection is nevertheless
worth further consideration.

HTTP redirects are a mechanism used by HTTP servers to inform a client
that further work needs to be done.  In some cases, the resource
client seeks has been moved, meaning the server may be informing the
client of the resource's new location. More interestingly, redirects
can be used to "string together"--or *pipe*, so to speak--a series of
HTTP requests, where each step provides a proper transformation of the
previous request's results to the next request.

In what follows, I will show that many contemporary uses of HTTP
redirects exhibit an analogous behaviour to UNIX shell commands.
Furthermore, I will argue that due to this use, a distinction between
redirection and safe redirections must be made.  Where a safe
redirection is one that is not reflected visually in the browser's UI,
allowing possible improvements to user experience.

<<Insert shell/redirect behaviour comparison here>>

Before we continue, we require a few definitions: We need distinguish
between a single redirect and a *transparent* redirect.  A transparent
redirect is a series of *n* redirects, for *n > 1*.  Since transparent
redirects may traverse one or more domains, we require a further
distinction between safe and unsafe transparent redirects.  A safe
redirect is a transparent redirect within a [trust domains, forests or
realms][3]. In other words, if the sequence of redirects follow a path
between trusted regions, then the sequence is considered safe.

How to implement safe redirect is beyond the scope of this post.  For
now, we assume that the infrastructure to perform them already exists.
The implementation details will be addressed in a future post.

Having now defined safe redirects, we can concentrate on their use.
Like existing sequences of redirects, safe redirects use one more
application servers to provide more complex client/server interaction,
while reducing coupling between application modules as well as
separate applications.  A typical example of these sequences can be
seen in many authentication steps required for modern web
applications.  For instance, some bank web applications require a
complex sequence of redirects to properly authenticate a user.

While it may comforting to know that such an institution is taking
such extensive precautions to secure personal data, being reminded
of it on every visit to the site provides little value.  That is: if
the only indication of the process is a series of redirects, which can
be seen to provide a portion of the authentication process, then it
may be of benefit to simply group the sequence of operations in to a
single unit.  Herein lies the benefit of a safe redirect.

By using a safe redirect, web applications can *hide* the sequence of
redirects from the end user, so as to reduce the amount of navigation
noise experienced by the client software.  Which is to say: if a user
needs to check their account balance, the login process should take
them from the username and password page and land them directly in
their account, rather than having the client software, i.e browser,
load and render intermediate pages, as defined by the login redirect
sequence.  The net effect of this change would be to eliminate the
need for any visual queues to a user about the nature of the
authentication process.

This change may be used to provide two key user experience
benefits. First, a sequence of redirects would apear as a single page
navigation.  And second, the return of back functionality of the
client could be used to undo *all* redirects in the last sequence,
rather than simply the last redirect of the sequence.  Meaning that if
a user is delivered to an account page, clicking the back button in
their browser would return them to the login page.


[1]: http://okmij.org/ftp/Computation/monadic-shell.html
[2]: http://www.w3.org/Protocols/HTTP/1.0/draft-ietf-http-spec.html
[3]: http://technet.microsoft.com/en-us/library/cc773178(v=ws.10).aspx
