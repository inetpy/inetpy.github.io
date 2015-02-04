---
title: 'From VIRL to IOU and Back'
author: Yandy
layout: post
banner_image: logo_cisco1.jpg
date: 2015-02-04 13:53:01
permalink: /2015/02/from-virl-to-iou-and-back
categories:
  - Networking
tags:
  - virl
  - iou
  - routing
  - cisco
  - study
  - ccie
---
When VIRL first came out everyone jumped on the bandwagon, including myself. Some of us have had it before it was officially released to the public. Cisco's VIRL is really a good piece of software, but it has its short comings.

<!--more-->

### VIRL 

If you're styding for CCIE or making quick config checks:

* The 15 router limi is not enough
* Most vendors require 20+ routers
* Resource intensive
	* Even when running only a few instances
	* My 16GB MBP struggles
* No L2 support yet (not a big deal for new CCIEv5)
* $200 for most (can be a bit high depending on usage)

If you're building out a POC type network, VIRL is definitely the way to go, since it was deisgned to run on x86 platforms there's less bug vs configuration contention.

VIRL does have:

* Support for many more instance types
	* NX-OS
	* IOS
	* IOS-XR
	* IOS-XE
	* This list will probably grow
* Built to run on x86 Platforms
* Pre-build topolgies and configurations with AutoNetkit
* Run Ubuntu instances to test end-to-end
* Much more

These are the reasons for a more *real world* approach to using VIRL. But if you're studying, or simply want to look up a command. Bringing up an instance can take a while depending.

### IOU

IOU is not publicly available, and don't ask if I have it, and can I share it. I will not share any IOU related files. This is simply about my preference when it comes to throwing something up quickly.

Unless there's some platform specific feature (IOS-XE, XR, NX-OS) IOU can be very helpful for quick configuration checks. They can run in mere minutes and basically any machine. 

Case for IOU:

* Barely any resource usage
* Quickly run on any Linux VM
	* Can run many instances 20+ with very little resources
* Loading and re-loading configs much quicker
* Instances start up much faster

The NOT so CASE for IOU:

* Can be very involved even after initial install
	* Require scripts and text files to form topology
	* Unless you use IOU-WEB type interface (I don't)
* Not easily accissible
* Updates hard to come by
* L2 support is minimal (extremely buggy)
* Again this is not really a big deal for me

### Summary

I still use both, but like I mentioned, if there's something that requires any resemblance to real world network. Then, I will bring up VIRL and configure it there as bugs are less frequent and can be validated much simpler. If all I need is a quick configuration check, or studying for the CCIE Lab, then my pick goes to IOU.

I have not tried the new GNS3, but hear there's support to running IOU based instances if you have access to the images.