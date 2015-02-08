---
title: 'Learn to Code'
author: Yandy
layout: post
date: 2014-04-04 13:13:02
image: coding_banner.jpeg
twitter_image: coding_banner.jpeg
description: Useful to code? I believe it is, and you should too ;)
summary: I took a few C++ and Visual Basic courses back in high shcool. Now, at that time, they didn’t teach the STL (Standard Template Library), I remember using conio.h allot, not really understading what or why at the time. Never did much with any language until recently since...
permalink: /2014/04/learn-to-code-part-1/
categories:
  - Coding
tags:
  - sdn
  - coding
  - programming
  - learn-to-program
  - learn-to-code
  - python
  - cpp
---
<hr>
![](http://ipyandy.net/images/coding_banner.jpeg)
<hr>

### Background

I took a few **C++** and **Visual Basic** courses back in high shcool. Now, at that time, they didn’t teach the STL (Standard Template Library), I remember using **conio.h** allot, not really understading what or why at the time. Never did much with any language until recently since.

The programs weren’t very useful, a console hangman game and a few other *“useless utilities”*.

### Not Just for SDN or Network Programmability

However, I’m glad that I took those classes, and here’s why. Software development practices teach you to take a much bigger problem or issue and break it down into smaller, simpler tasks.

These tasks are broken down into functions or methods, smaller segments of code that perform a specific task for the overall problem you’re solving.

#### Examples

We all know the show version command, and all the information it returns. Let’s say we wanted to grab only certain subsets of that information. How would we do this?

We can write a long complicated file, that does something like this (examples are in **Codinglish**) my term for Coding in English.

This can get very tedious, and error prone, fixing one part can affect another because everything is tied. Now if, we wrote smaller bits of code (functions).

This is a very simplistic example (not actual code), the code will be much more complicated. But still breaking it down to find individual parts, instead of hundreds of lines to find single part and hoping that a change later will not break it.

### So What?

So what does this have to NOT do with network programming? It seems like that’s exactly what it is. You would be right, it is an example based on network information to use something we’re familiar with.

The idea however is to break tasks, whatever that may be. Troubleshooting an OSPF adjacency? Break that down into separate functions or methods.

* Physical / L2 link ok?
	* cabling
	* sfp
	* duplex
	* speed
* Ping the neighbor?
	* response?
	* no response?
	* one side?
		* ACLs on the interfaces?
* Subnet match?
	* different?
	* same?
* Network type match?
	* point-to-point vs broadcast vs non-broadcast vs.. you know
* Authentication match?
	* plain text?
	* md5
	* no auth
	* both sides of the link?
* … and so on

By breaking these problems down into smaller parts, instead of trying to tackle the entire thing at once. Looking at a configuration for hours to miss the simplest of errors, such as 255.255.255.0 and 255.255.254.0. Because your eyes and mind are tired from handling the problem as a whole.

### Summary

Recently, I’ve been sharpening my coding skills with Python. Why Python? Well, because it’s an awesome language that’s why. Not going to get into, this language vs that language here, that’s just the one I chose.

Taking on an open-source project in Python reminded me of what a real good concept this is. Break things down, as small as possible without losing too much information, or making it more complicated by adding more than you need.
