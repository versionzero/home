---
layout: post
title: "Towards a working Raspberry Pi in-car computer"
date: 2013-02-24 19:52
comments: true
categories: [linux,pi,usb,bluetooth]
---

Ever since I purchased my latest vehicle, I've been working on a
little side project to interface directly with the vehicle's on-board
computer.

The task itself is pretty straight forward: buy device capable of
plugging in to the vehicle's ODB2 port, and run some free or
inexpensive Android app to gather the information.

Unfortunately, the above process requires a great deal of user
intervention.  First, you must always plug and unplug the ODB2 device.
Second, you must either have an Android device on your person or in
the vehicle capable of running the required software or a computer
with a variety of command-line tools installed.  The latter is
impractical, and the former is error prone (I'll forget to start the
app, take the readings, etc.) and tedious (The data must later be
extracted from the app manually, or a web-service must be made
available to push the data to-- which also requires a data plan, if
the car is not always in the proximity of a wifi access point).

Raspberry Pi to the rescue.

The [Raspberry Pi](http://www.raspberrypi.org) is a small, inexpensive
low-power computer machine.  It was originally developed with teaching
in mind, but sparked an enormous hobby market.  The Raspberry Pi, or
raspi for short, is available in two models: Model A is $25 the Model
B is $35. The differences between the models is nominal: the more
expensive one has more USB ports, more RAM and an Ethernet port.  As a
developer, the Model B is preferable, since it provides easy extension
through USB and trivial network access via ethernet, not to mention
more room to write sloppier prototype code in the roomy 512MB main
memory.  The Model A would be best suited to production or permanent
projects, post development.  (Not true in all cases: having network
access can be critical for some applications-- though a USB wireless
dongle may mediate this issue.)

I purchased two of the $35 Model Bs, for a separate project, but
quickly found that they would be a good fit for a variety of computing
projects.  One of these projects ended up being the computer for my
vehicle.

The original intention of car computer was to keep track of details
that were important to me, but slightly time consuming and tedious.
One simple motivating example was tracking fuel economy.  It's a
simple calculation to make, record and compare to previous trips.  But
why bother doing this work by hand if it will eventually end up in a
computer, if the computer itself can gather and tabulate the data
automatically.  It's hardly an earth shattering improvement, but led
to further ideas.  For instance: it would be great to know *where*,
*when* and *why* the vehicle experiences the worst fuel economy.  By
taking additional data samples, like location, speed, temperature,
time, and a variety of engine performance metrics, a very clear
picture of how a vehicle performs will emerge.

Towards these ends, I've been configuring a raspi to read from a
Bluetooth ODB2 adapter and stash the information until it can be
transferred to my home network for processing.

The idea is simple: poll the ODB2 adapter periodically for all data it
can return.  Simultaneously poll a GPS unit and stash the correlated
data locally on the raspi device.  If and when my home network is
within range, join the wireless network and transfer the collected
data for processing.

*more on the way*
