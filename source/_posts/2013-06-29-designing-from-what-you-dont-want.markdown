---
layout: post
title: "Designing from what you don't want."
date: 2013-06-29 09:04
comments: true
categories: design
---

I don't know about you, but I very rarely know what I want.  Sometimes
this is because there are too many choices and I can't make up my
mind.  Other times, it's because I am genuinely too ignorant to know a
good thing when I see it.

This all happend sometime after my teenage years. At least then, in
spite of knowing less than I do now, I knew exactly what I wanted.
Actually, I should qualify that: I strongly (arrogantly?) believed I
knew what I wanted, because I knew _everything_. I miss the confidence
of youth.

Anyway, this is not a post about the constant uncertainty of adult
life.  There are plenty of those around to last us well in to the
introduction of the next species.

This post is about designing from what you _don't_ want.

So the idea is pretty simple: we all have years of experience doing
things we have not wanted to do.  Some of these activities have been
less scaring than others.  But again, this is not about emotional
states, well not entirely, this is about designing new things and
improving old ones.  What we aught to be doing is designing away the
things we do not like doing: it's much easier to identify particular
behaviour we find less than desirable than it is to fairly itemize the
things which we find desirable.

Ok, you might ask, how does this differ from the way we do things
already?  We all fix bugs, add convenience features, etc.. Don't these
satisfy the above design criteria?

Yes and no.

Yes, fixing a bug does remove an irritant, and adding new features to
do all sorts of wonderful things is great.  But with new features come
new irritants.  Our complete design philosophy is to fix broken things
and add new things. Features sell, bug-fixes retain. This is not a
state we should be satisfied with: just because it works, doesn't mean
it ain't broke.

What I'm proposing is subtlety different: _remove_ unnecessary parts,
steps, etc..  This is hardly a novel idea in and of its self, we all
learnt it as a child.  Or, at least, we brushed up against it.  That
punishment was not desirable thing.  Thus, could either behaved well,
or hide our punishable behaviour in a better way.  The same principle
applies: rather than enduring the process because of the outcome,
change the way it is done.

So it's at this point where we should stop and look at an example,
lest we allow the above sound like self-help session.

I hate keeping track of hours. Ok, not hate, that's too string: I just
find it tedious. I tend to either postpone doing it, or, because I've
become so adept at ignoring it, I simply forget to do it.  But I can't
get rid of tracking hours.  It's part of my job, and it does help in
the long run: it lets us determine a better guess as to how long new
step will take to enact. (I don't get payed by the hour, so even that
is not an incentive.)

So how do I reconcile my distain for the activity and the benefits
that can be derived from it. I could, as my mother might say, learn to
_grin and bear it_; or, I could find a way to minimize the time that
is needed to be spent on that particular activity.

For me, the answer is simple: find or make new a tool.

I spend most of my working day using revision control tools or
rhythmically molesting my keyboard in an editor. Ideally, the new tool
would fit in this workflow.

If I was being honest, this is what I'd want:

```
git flow feature start something-cool
// hack, hack, hack...
git add cool.c
git ticket 43 +4h
// did something horribly wrong, how am I still allow to write code?
git ticket 43 +1.5h
git ticket 43 +5h
git ticket 43 +20m
// finally, it's fixed
$ git flow feature finish something-cool
```

Now, I'm married to the sytax, I'd prefer that an arbitrary ticket
number so associable with an arbitrary branch (including a ticket
branch).  Meaning the above could be simplified further:

```
git flow feature start something-cool
gut ticket associate 43
  // => git-ticket: Active JIRA ticket 43
git ticket 4h
  // => git-ticket: Added 4 hours to JIRA ticket 43
git ticket 20m
  // => git-ticket: Added 20 minutes to JIRA ticket 43
$ git flow feature finish something-cool
  // => git-ticket: Active JIRA ticket 43
  // => git-flow: ...
```

Further, since it's likely we will want to support more than one
project at a time.  This means we might run in to a situation where a
seperate, unrelated project needs to be updated with respect to hours.
Support for this case would take the following form:

```
git flow feature start something-cool
gut ticket associate 43
  // => git-ticket: Active JIRA ticket 43
git ticket 4h
  // => git-ticket: Added 4 hours to JIRA ticket 43
git ticket 20m 545
  // => git-ticket: Added 20 minutes to JIRA ticket 545
$ git flow feature finish something-cool
  // => git-ticket: Active JIRA ticket 43
  // => git-flow: ...
```

Notice that while we update can update a ticket at anytime, it does
not mean we change the active ticket.  A sample use might be to do
something to a remote system at the command line, then adding time to
the ticket directly.  This bipasses the need to set the effemeral
ticket active: still, this is seems like the correct behaviour for
many cases.
