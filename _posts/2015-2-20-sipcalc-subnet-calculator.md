---
title: Using sipcalc The Great Subnet Calculator
author: Yandy Ramirez
layout: post
date: 2015-02-20 11:20:01
image: osx-001.jpg
twitter_image: osx-001.jpg
description: In the CCNA days we all had to learn how to subnet and work with binary. This was either on paper or in your head if you were quick enough to do it. This really is an invaluable skill when speaking, talking IP design and more. When someone throws out a lets just use the 10.98.0.0/12 for this site. It's good to know that it wouldn't quite work (bit boundaries and all)
summary: In the CCNA days we all had to learn how to subnet and work with binary. This was either on paper or in your head if you were quick enough to do it. This really is an invaluable skill when speaking, talking IP design and more. When someone throws out a lets just use the 10.98.0.0/12 for this site. It's good to know that it wouldn't quite work (bit boundaries and all)...
permalink: 2015/02/sipcalc-subnet-calculator/
categories:
  - Networking
tags:
  - sipcalc
  - using sipcalc
  - subnetting
  - subnet calculator
  - osx
  - os x
  - mac os x
  - os x subnet calculator
  - linux
  - ubuntu
  - bsd
---
<hr>
![](http://ipyandy.net/images/osx-001.jpg)
<hr>

In the CCNA days we all had to learn how to subnet and work with binary. This was either on paper or in your head if you were quick enough to do it. This really is an invaluable skill when speaking, talking IP design and more. When someone throws out a lets just use the 10.98.0.0/12 for *this* site. It's good to know that it wouldn't quite work (bit boundaries and all). 

Though doing an IP design/scheme for an Enterprise entirely in your head is not feasible and extremely error prone (things are always error prone). Enter **sipcalc** a nix (linux, bsd, os x) command-line utility that's fast and supports IPv6, yes that's right IPv6 and all.
<!--more-->
<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<!-- ipy_responsive_2_text -->
<ins class="adsbygoogle"
     style="display:block"
     data-ad-client="ca-pub-2031545302097188"
     data-ad-slot="7544446295"
     data-ad-format="auto"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>

### What's all the Fuss?

I pride myself in my *subnetting skillzz* and all, but sometimes it's great to have a companion to keep you honest (we all make mistakes). Lets say we're working on a specific region's IP subnet scheme, using the 10.48.0.0/12 and we ned to break that down to a number of sites. What does this /12 look like? What range does it have? Yes, some of you already did it in your head, it's simple enough, but this is a simplistic example (so hush!).

*sipcalc [subnet]/[bits]* will give you an answer.
{% highlight bash %}
$ sipcalc 10.48.0.0/12
-[ipv4 : 10.48.0.0/12] - 0

[CIDR]
Host address		- 10.48.0.0
Host address (decimal)	- 170917888
Host address (hex)	- A300000
Network address		- 10.48.0.0
Network mask		- 255.240.0.0
Network mask (bits)	- 12
Network mask (hex)	- FFF00000
Broadcast address	- 10.63.255.255
Cisco wildcard		- 0.15.255.255
Addresses in network	- 1048576
Network range		- 10.48.0.0 - 10.63.255.255
Usable range		- 10.48.0.1 - 10.63.255.254
{% endhighlight %}

With a simple command we know subnet mask, network bits, usable range and even a wildcard mask for creating access-lists. What if this *region* had to major sub-regions of it's own, and we wanted to split those into /14s for example? 

*sipcalc [subnet]/[bits] -s [split-size-bits]*
{% highlight bash %}
$ sipcalc 10.48.0.0/12 -s 14
-[ipv4 : 10.48.0.0/12] - 0

[Split network]
Network			- 10.48.0.0       - 10.51.255.255
Network			- 10.52.0.0       - 10.55.255.255
Network			- 10.56.0.0       - 10.59.255.255
Network			- 10.60.0.0       - 10.63.255.255
{% endhighlight %}

Now each one of these sites within the first /14 will be using a /17 for their inidiviual subnets. We can find that out the same way.

{% highlight bash %}
$ sipcalc 10.48.0.0/14 -s 17
-[ipv4 : 10.48.0.0/14] - 0

[Split network]
Network			- 10.48.0.0       - 10.48.127.255
Network			- 10.48.128.0     - 10.48.255.255
Network			- 10.49.0.0       - 10.49.127.255
Network			- 10.49.128.0     - 10.49.255.255
Network			- 10.50.0.0       - 10.50.127.255
Network			- 10.50.128.0     - 10.50.255.255
Network			- 10.51.0.0       - 10.51.127.255
Network			- 10.51.128.0     - 10.51.255.255
{% endhighlight %}

You can probably see a pattern here, break each site into /20s, /22s and break those further down per users or requirements. There's other options, such as getting just a wildcard for a specific subnet mask.

{% highlight bash %}
 sipcalc 255.255.240.0 -w
-[ipv4 : 255.255.240.0] - 0

[WILDCARD]
Wildcard		- 255.255.240.0
Network mask		- 0.0.15.255
Network mask (bits)	- 12
{% endhighlight %}

### Bonus IPv6

I mentioned that **sipcalc** does IPv6, and most of the same options that work on IPv4 work with IPv6. The difference for splitting a subnet/prefix is just adding a capital (-S) instead of the (-s).

{% highlight bash %}
$ sipcalc 2001:1::/32 -S 34
-[ipv6 : 2001:1::/32] - 0

[Split network]
Network			- 2001:0001:0000:0000:0000:0000:0000:0000 -
			  2001:0001:3fff:ffff:ffff:ffff:ffff:ffff
Network			- 2001:0001:4000:0000:0000:0000:0000:0000 -
			  2001:0001:7fff:ffff:ffff:ffff:ffff:ffff
Network			- 2001:0001:8000:0000:0000:0000:0000:0000 -
			  2001:0001:bfff:ffff:ffff:ffff:ffff:ffff
Network			- 2001:0001:c000:0000:0000:0000:0000:0000 -
			  2001:0001:ffff:ffff:ffff:ffff:ffff:ffff
{% endhighlight %}

Kept it short, because well, IPv6 prefixes can easily take my entire buffer on-screen :). 

##### Quick Install

**Ubuntu / Debian**
{% highlight bash %}
sudo apt-get install sipcalc
{% endhighlight %}

**Redhad / Fedora / (Redhat based OS)**
{% highlight bash %}
sudo yum install sipcalc
{% endhighlight %}

For other OSe's look up the documentation or download from source and compile (Google it!).

### Summary

It's a pretty handy utility to keep at hand, not sure if there's a windows version, never looked. But if you're interested in more features **man sipcalc** is always a good option.
