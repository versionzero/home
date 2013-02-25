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
low-power computing machine.  It was originally developed with
teaching in mind, but sparked an enormous hobby market.  The Raspberry
Pi, or raspi for short, is available in two models: Model A is $25 the
Model B is $35. The differences between the models is nominal: the
more expensive one has more USB ports, more RAM and an Ethernet port.
As a developer, the Model B is preferable, since it provides easy
extension through USB and trivial network access via ethernet, not to
mention more room to write sloppier prototype code in the roomy 512MB
main memory.  The Model A would be best suited to production or
permanent projects, post development.  (Not true in all cases: having
network access can be critical for some applications-- though a USB
wireless dongle may mediate this issue.)

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

## Configuration

### USB Hub Configuration

The first problem I ran in to was that the cheap USB Bluetooth dongle
I purchased was not well received by my raspi.  Fortunately, like many
things, I was not the only one who had this issue.  A quick web-search
away turned up the [solution](
http://raspberrypi.stackexchange.com/questions/1886/what-kernel-parameters-are-available-for-fixing-usb-problems):
the USB hub's speed had to be turned down to the USB 1.1 standard, as
the 2.0 standard was known to no play well with some cheap devices.

The fix was simple, once I found it, and only involved changing a few
boot variables the kernel uses to initialize hardware on boot.  In
particular, I changed:

* `dwc_otg.speed`: 1 will limit USB speed to full speed 12Mbps (USB 1.1).
* `dwc_otg.microframe_schedule`: 1 (default now) This should fix the
  error when too many periodic endpoints are present.

### Bluetooth Configuration

Having no documentation on the ODB2 sensor I purchased, I used the
`sdptool` to tell me a little about the device.

``` bash
$ sudo sdptool records 00:19:5D:27:12:6E
...
Service Name: SPP Dev
Service RecHandle: 0x10002
Service Class ID List:
  "Serial Port" (0x1101)
Protocol Descriptor List:
  "L2CAP" (0x0100)
  "RFCOMM" (0x0003)
    Channel: 1
Language Base Attr List:
  code_ISO639: 0x656e
  encoding:    0x6a
  base_offset: 0x100
```

It's not a hugely informative dump of information---in fact it's
downright cryptic---but it does provide us with the protocol and
channel to connect with.

By configuring `/etc/bluetooth/rfcomm.conf` correctly, we get a device
`/dev/rfcomm0`.

``` bash
rfcomm0 {
  bind yes;
  device 00:0D:3F:45:DF:8A;
  channel 1;
  comment "ODB2 Sensor";
}
```

Then we only need to restart the bluetooth service:

``` bash
$ sudo systemctl stop  bluetooth.service && \
  sudo systemctl start bluetooth.service
```

Eventually one could use the command:

``` bash
$ sudo rfcomm bind 0 00:15:A0:7A:90:F2 1
```

Where the 0 is the device suffix, the hardware address is that of the
ODB2 sensor and the 1 is the channel we want to connect on.  We should
now see the RFCOMM device available for use:

``` bash
$ ls -l /dev/rfcomm0 
crw-rw---- 1 root uucp 216, 0 Jan  1 01:14 /dev/rfcomm0
```
