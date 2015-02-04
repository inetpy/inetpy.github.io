---
title: The Annoyance of AnyConnect Mac Client
author: yandy
layout: post
permalink: /2012/11/the-annoyance-of-anyconnect-mac-client
dsq_thread_id:
  - 1247404336
categories:
  - Mac OS X
  - Networking
tags:
  - annyconnect os x
  - anyconnect
  - anyconnect mac
  - cisco anyconnect
  - mac
  - os x
---
### AnyConnect Mac

I find it annoying that sometimes, the list of remembered connections (which is just one), turns out to be a VPN you never use, or only used once. This is a problem I find only on the Mac AnyConnect client, (never tested the Linux version). The problem is, even if you establish another connection, when the client is re-opened it goes back to that original profile link.

I was trying to figure out how to get rid of that connection when you open up the client, and have it remember the **last** connection.

What worked for me, was basically deleting all the files under */opt/cisco/anyconnect/profile/*.

<pre lang="plain">sudo rm /opt/cisco/anyconnect/profile/*
</pre>

One of the profiles in there can be edited and blanked out, but it’s allot easier to just delete the files. It also doesn’t seem to brake any functionality, this should allow the AnyConnect client to remember the last VPN connection you established.