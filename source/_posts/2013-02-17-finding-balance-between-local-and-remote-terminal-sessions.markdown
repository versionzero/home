---
layout: post
title: "Finding balance between local and remote terminal sessions"
date: 2013-02-17 08:27
comments: true
categories: [bash,ssh,balance,load-balancer,computer]
---

## Introduction

Recently, a number of our externally visible school machines were
compromised. It is not clear the extent of the attacks effect, but
from some of the details I did get, it sounds like the work of an
amateur. Either way, the point of this article is not to speak to this
issue directly, but rather focus on the inconvenience of having some
of the host names you are accustomed to having access to suddenly
disappear. To be clear, this is not an article complaining about
having to change the hostnames we use on the command line, but ones
that we have embedded in scripts and configuration files.

I know I am not the only one who was caught by this, but it is not
clear if others where as throughly irritated by the change. I do a
great deal of work remotely and, as such, I am highly dependant on the
stability of the set of host names I use. So, in order to protect
myself from any similar future mishaps, I made a few changes to the
way I work.

It is not clear if my approach is particularly novel, but I've not
seen it listed elsewhere, so I thought I would document it for others
to try. It occurred to me that I could easily bypass these issues by
simply running a local load balancer that balances over all the remote
school hosts. There are a number of issues that crop up using this
approach, and I will try to address them as thoroughly as possible.

## Finding Balance

First things first: the software. I used an appropriately named tool
[Balance](http://www.inlab.de/balance.html). As the name implies, it
is a load balancer--and a very simple and successful one at that. I
used my trusty old package manager to install it, but building it from
source would be equally appropriate.

The syntax for Balance is very simple, so we can discuss it very simply:

``` bash
$ balance 2222 linux1.example.com:22 linux2.example.com:22 linux3.example.com:22 
```

This tells Balance that connections to the local port 2222 should be
forwarded in an alternating fashion to port 22 on
`linux1.example.com`, `linux2.example.com` and
`linux3.example.com`. Furthermore, it tells Balance implicitly to run
in the background.

Balance can be told to bind to the local port explicitly, via the -b
option, but I generally only have one network adapter active at any
one time, so calls to `bind()` will fail because it will be attempting
to bind to the same local port on the same IP address. There are ways
to get around this limitation, by
[binding multiple IP addresses to a single network adapter](http://www.xenocafe.com/tutorials/linux/redhat/bind_multiple_ip_addresses_to_single_nic/index.php). 
I chose not to take this route, however, as it seems needlessly
complex for my needs. Anyway, moving on.

## Automation

Now that we have established how to balance over a number of remote
hosts, we would probably like to automate the whole thing. For my
purposes, I added something similar what follows to my shell
profile. I have removed my comments to improve readability, but I will
expand on the behaviour following the code.

We start by choosing a port >1000. Doing this means we do not need to
run our scripts with a super user privileges. The main benefit of this
is simplicity. That being said, I can think of no good reason for
choosing a sub-1000 port, except that we would not be required to
explicitly state which port to use when using `ssh` (more on this point
later). Furthermore, using a sub-1000 port would likely require
additional security considerations that I don't care to think about
right now.

```
LOCAL=2222
REMOTE=22
```

Next we build a list of all the destination servers we will "load
balance" over. Balance will just round-robin over them: if one is
down, it will move on to the next. This makes it simple for writing
scripts that depend on a host name: i.e. there is no need to write
ugly fail-over code. Note that I have listed the server names in
decreasing order. I do this because it seems that most people that
work remotely tend towards using the lower numbered hosts first. So in
some sense this is just a naive heuristic for finding unloaded
systems, without foreknowledge of the systems' load.

``` bash
SERVERS=
for i in {1..3}; do
    SERVERS="linux${i}.example.com:${REMOTE} ${SERVERS}"
done
```

The final step is to start Balance. But before we try to start the
Balance, we should check if it is already running. If we do not do
this, Balance will attempt to bind to a port that is already bound,
and die a very ungraceful and uninformative death:

```
bind(): Address already in use
```

We also need to check that the instance of Balance is ours. If it is
not, then we will need to spawn our own. The procedure bellow is not
fool-proof, mind you, but it should work for our limited purposes. The
solution can always be made more robust as the need arrises.

``` bash
PID=`pidof | grep ${USER} | grep balance | awk '{ printf $2 }'`
if [ "${PID}x" == "x" ]; then
    balance ${LOCAL} ${SERVERS}
fi
```

So that is it for getting Balance up and running. Once we have spawned
Balance, we can check on it fairly simply using ps:

``` bash
$ ps ax | grep [b]alance
 80989 s001  S      0:00.04 balance 2222 linux3.example.com:22 linux2.example.com:22 linux1.example.com:22
```

The `[b]alance` notation stops `grep` from matching itself. This trick
works because the command line for `grep` no long contains the pattern
being matched.

## Usage

Now we would like to actually make use of our new load balancer. We
could use the new port and localhost name directly with ssh, but this
can be a tad cumbersome, and easy to forget to do:

``` bash
$ ssh localhost -p 2222
```

It also suffers from a far worse problem. This command line will work
the first time, but will likely fail on subsequent invocations. The
reason for this should be clear from the error message:

```
$ ssh localhost -p 2222
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
@    WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED!     @
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
IT IS POSSIBLE THAT SOMEONE IS DOING SOMETHING NASTY!
Someone could be eavesdropping on you right now (man-in-the-middle attack)!
It is also possible that the RSA host key has just been changed.
The fingerprint for the RSA key sent by the remote host is
33:ab:39:23:6f:4a:d9:86:bb:de:fe:2d:1f:6a:35:4d.
Please contact your system administrator.
Add correct host key in /home/user/.ssh/known_hosts to get rid of this message.
Offending key in /home/user/.ssh/known_hosts:42
RSA host key for [localhost]:2222 has changed and you have requested strict checking.
Host key verification failed.
```

What has happend is that we have moved on to a different host in the
list of candidate hosts we defined above. So the warning is
effectively a red-herring: in the general case, it is not that the
host's key has changed, but that we are looking at a new host, but
considering it the same host as the previous one.

We can fix this by telling ssh to disable strict host checking for the
localhost:

```
$ ssh localhost -p 2222 -o NoHostAuthenticationForLocalhost=yes
```

This command will always work, but it is a rather unruly beast to type
each time we want to login remotely. We could of course just write an
alias for our shell, but this limits, in some sense, the flexibility
of our configuration. Thus, we turn to the ssh_config file for further
inspiration.

## SSH Configuration

I typically have a number of different user accounts on various
different systems. I will not get in to the virtues of working this
way, as I am sure that would take up as much space as this article
has, and it is not entirely clear to me that many people would in fact
agree that this is a good idea. We shall just settle with the fact
that I do it, and I need my environment to allow me to work on many
accounts transparently.

The ```ssh_config``` file allows a user to define a set of hosts, and
the options to go with them. We could, for instance, have two users
**user1** and **user2** configured to use the same set of hosts:

```
Host user1
    HostName localhost
    User user1
    Port 2222
    NoHostAuthenticationForLocalhost yes

Host user2
    HostName localhost
    User user2
    Port 2222
    NoHostAuthenticationForLocalhost yes
```

Note that we actually use the user names in place of the host
names. This makes connecting to the load balanced hosts very
simple. For instance, to log as **user1** we would type the following:

```
$ ssh user1
```

Which is a far cry from the command line we required just a few
minutes before. We can do the same for **user2** and we could also add
more hosts, and more options, but I will save that all for another
time.

## Making the Leap

None of my office machines are public facing, so they require a second
hop once I've logged in to the public machines. This can get a little
tiresome to type if you end up working remotely a lot, like I do.

To save myself a few keystrokes, I add the following entry to my
`~/.ssh/config` file:

```
Host user3
    HostName localhost
    User user3
    Port 2222
    NoHostAuthenticationForLocalhost yes

Host office
    HostName private1.example.com
    User private1
    Port 22
    # I actually really just want to hop from the public facing linux
    # machines to my office machine
    ProxyCommand ssh -q user3 nc -p0 %h %p
```

Where `nc` is a tool by the name netcat, a very hand network
listener/sender/swiss army knife. Now, thanks to the simple
configuration above, all I need to write is:

```
$ ssh office
```

and I'll be connected directly to the workstation in my office.

## Security

There is one last issue that I have been silently ignoring. The astute
reader may have noted that we can no longer detect legitimate
person-in-the-middle attacks, because we have explicitly told ssh not
to authenticate host identities. I purposely ignored this issue for
various reasons, arguably the most important one, to me, is that
accounting for it would greatly complicate the whole article. Time
permitting and cleverness permuting, I will try to address this
shortcoming in a future article.

## Code

As per usual, the code for all my work is freely available online. If
you want to look at my implementation of these ideas in particular,
point your browser at the following two pages:

* [SSH Load Balancing](https://github.com/versionzero/dotfiles/blob/master/profile.d/90ssh-balancer.sh) ([raw](https://github.com/versionzero/dotfiles/raw/master/profile.d/90ssh-balancer.sh))
* [SSH Configuration File](https://github.com/versionzero/dotfiles/blob/master/ssh/config) ([raw](https://github.com/versionzero/dotfiles/raw/master/ssh/config))

