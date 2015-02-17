---
title: 'Device Naming Conventions - What's in a Name?'
author: Yandy
layout: post
date: 2015-02-17
image: device_hostname.jpg
twitter_image: device_hostname.jpg
description: Choosing a device hostname seems trivial to say the least. This is not always the case when discussing it in the open.
summary: "Choosing a device hostname seems trivial to say the least. However, from multiple design meetings, this is a topic that tends to drag on. Everyone has a preference, and opinion or just set in the ways *we've done* this way for a long time."
permalink: 2015/02/device-naming-conventions/
categories:
  - Design
tags:
  - hostname
  - device-hostname
  - switch-hostname
  - switch
  - routers
  - network-naming
  - device-name
  - cisco
  - cisco-hostname
---
<hr>
![](http://ipyandy.net/images/device_hostname.jpg)
<hr>
Choosing a device hostname seems trivial to say the least. However, from multiple design meetings, this is a topic that tends to drag on. Everyone has a preference, and opinion or just set in the ways *we've done it* this way for a long time and so on with other excuses.

### What's in a Name?

I've seen names so simple that it was nearly impossible to tell the function by just looking at the name. This excludes the default **Router** or **Switch** that yes, I have seen in production. But things like, sw-gig1. Is this a **Core** switch? An **access-switch**? In this case it was a 4948 DC top-of-rack switch. 

There are also hostnames that are too long and complex that you just give up trying to figure out what it means. If you have to look up a code table in a database or excel sheet, it probably isn't a good naming convention (with some exceptions), names like **BEEIMA04BTEN** provide no immediate value to what you're working on.

#### Simplicity over Complexity

KISS (Keep it Simple Stupid) I'm a big believer in this, keeping simplicity at the core. But how do you express function with simplicity? Can your naming scheme be too simple? Of course it can, one of the examples above shows it. But this was simplicity without thought or just plain lazyness which happens.

### Lengths and Preferences

The name should be long enough to tell you, "I know exactly what this" by just looking at it. But not so long that you need double retina resolution to keep every other output or command from wrapping lines.

#### Geographical Locations

Should you put a hint as to what country, city or town the devices are located? Much of the standard is to put a designation of closest airport code, or simply closest international airport code. Such as **MIA-[REST-OF-NAME]** or **[REST-OF-NAME]-MIA**. Where you put this designation really doesn't matter. Keep it a standard and make sure that it's followed. I prefer in the beginning of the name, but it's personal preference at this point.

#### Function

What about intent for switch or router function? Is this a campus access switch or a data center leaf switch? Is it a WAN router or an Internet facing edge router? Would you choose **MIA-AS-[REST-OF-NAME]** where **AS** is access switch? I prefer to keep a two to four letter designation for function such as the one above. Can also go with **MIA-INET-[REST-OF-NAME]** for Internet edge router.  Here's a few quick points that I've followed.

* Campus Switches:
	* AS - Access Switch
	* DS - Distribution Switch
	* CS - Core Switch
* DC Switches:
	* TOR - Top-of-Rack Switch
	* AGG - Aggregation Switch (for non-leaf-spine)
	* DCS - Core Switch (for non-leaf-spine)
	* LEAF - Pretty obvious (Leaf)
	* SPNE - Spine Switch
* Routers
	* WAN - Again obvious WAN router
	* INET - Internet edge router
	* VG - Voice Gateway (I feel dirty mentioning voice ;) )

And so on, this doesn't cover everything, just something that's been kind to me over the years when naming things. 

#### Model Designation

This is a tricky one, because it can begin to take the hostname and increase it tremendously its length. The model designations I prefer to use are short such as **N7K** or **N5K** for Cisco Nexus 7000 and 5000 respectively. How about Catalyst switches? Again, keeping it short is my preference, going with **3560** or just **C3K** for some for of Catalyst 3xxx Series. The model is easy enough to decipher in a **show version** output. Just having an idea of what type of switch you're working on should be enough to avoid major CLI mistakes.

#### Physical Location

When it comes to physical locations, again keep it simple. I've seen hostnames have [data center - rack - space in rack] attached to them. This is adding complexity and unnecessary length. Drilling down that far into the location can easily be added to the **SNMP Location** string. Just adding an **IDF** or **MDF** location should suffice. Even adding **DCA** or **DC1** for data center within the geo location already added to the hostname should also suffice. 

How about where IDFs are numbered the same throughout out buildings? Adding a building code designation which many have as short as three characters can be enough. Such as **MIA-AS-C3K-XYZ-IDF1-[REST-OF-NAME]** can be more than enough.

#### Separating Multiple Switches

How about serial designation? or in other words **switch-1** and **switch-2** vs **switch-a** and **switch-b**? This one is completely open, they all work from what I've seen. My preference again, goes towards the numbering. Mostly it's an aesthetics thing, just prefer the numbering scheme.

#### Full Example

Just to wrap it up, here's some examples of how my preference for naming schemes look. Take it and modify it, tare it down or share your own views in the comments.

##### Examples

Can you guess what each is from just looking at the name?

MIA-WAN-A1K-DC1-1
DFW-PE-A9K-DC2-1
JFK-DCS-N7K-DC1-1

### Summary

Do you agree with this? Do you have your own preferences? I'm sure you do! Share them in the comments or drop me an [email][1] and I'll share thoughts in an updated post.

[1]: mailto:yr@ipyandy.net