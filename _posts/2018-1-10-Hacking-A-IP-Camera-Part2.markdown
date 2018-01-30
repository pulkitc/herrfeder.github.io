---
layout: post
title: Hacking An IPCamera Part2
date: 2018-1-10 0:0:0 +0800
categories: embeddedsec
tags: iot hacking hardware ARM
img:  
---

As I assume, that the npc binary holds the gSoup service, I will try to emulate it for easier debugging and reversing. 
Trying to emulate the npc service:
```
sudo chroot . ./qemu-arm-static -g 2222 ./bak/npc/npc &
```
Will result in this errors:
```
open  hi_gpio device error
open  hi_gpio device error
open  hi_gpio device error
open  hi_gpio device error
Unsupported ioctl: cmd=0x0003
fgCmosWorkUp: error:  regctl ioctl fail 
```
Afterwards execution will stop. It tries to access the gpio pins and triggers some sysexit.
To set up my environment I will try to execute a SSH server on the IPCam to enable debugging via gdbremote on the IPCam.
Before doing this I have to do some further reversing of this binary to hopefully reveal some secrets about it's communication.

Searching for "soap" strings in the binary only matches occurences in the .text sector. So I have to find out, when are this strings referenced.



To be continued...
