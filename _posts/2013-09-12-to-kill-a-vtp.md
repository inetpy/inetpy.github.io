---
title: To Kill a VTP
author: yandy
layout: post
image: VTP_VLAN.jpg
twitter_image: VTP_VLAN.jpg
description: "VTP is the Devil! Or is it?"
summary: The Devil! Yes, VTP is not the Devil itself, but it very well could be. I understand the "protect it" or make sure you know what you're doing arguments. Those are all fine and dandy, and the fact that you can't use VLANS > 1005 is not even the reason I hate it...
permalink: /2013/09/to-kill-a-vtp
dsq_thread_id:
  - 1752870234
categories:
  - Networking
tags:
  - 6500
  - catalyst
  - catalyst-6500
  - cisco
  - switching
  - vtp
---
The Devil! Yes, VTP is not the Devil itself, but it very well could be. I understand the "protect it" or make sure you know what you're doing arguments. Those are all fine and dandy, and the fact that you can't use VLANS > 1005 is not even the reason I hate it.

### Why Then?

Well, yes, you can't use VLANs > 1005 in the typical way, but you can't use VLANs < 1005 in subinterfaces either. Not on the 6500 platform running Sup-720-10G at least. This is what you get if you try using a VLAN < 1005 on a sub-interface.

<pre lang="shell">Core01(config-subif)#encap dot1Q 501
Command rejected: VLAN 501 cannot be allocated. VLANs 1-1005 are VTP VLANs
VTP mode is client or server and must be changed to Transparent/Off to use VLANs 1-1005
</pre>

You don't need to create VLAN, lets say 2501 to use int in a sub-interface. But, this limits flexibility in what you want to design. 

Of course, we know that trying to create a VLAN > 1005 while runing VTP will fail worse than Miley Cyrus trying to *twerk* (that's a word now). 

Kill VTP!! and ban anyone that uses it from networking (ok, an extreme, but I really don't like it).