---
title: The OSPFv3 Router LSA
author: yandy
layout: post
permalink: /2011/05/the-ospfv3-router-lsa
dsq_thread_id:
  - 2479694919
categories:
  - Networking
tags:
  - ccie
  - OSPF Database
  - ospfv3
  - router lsa
  - cisco
  - ospfv3 router lsa
  - cisco ospf
  - cisco ospfv3
---
### Overview

In most cases, OSPFv3 behaves identical to good old OSPF which we all love; right? Except for some minor differences in certain LSAs, some new one’s and others that are the same LSA, but behave a bit different. The purpose of this article is to try and cover the firs three of these LSAs. The Router LSA, which has the same name as it’s OSPFv2 counterpart. The Intra-Area-Prefix LSA, which is new in OSPFv3. Last but not least, the Link specific LSA, which is also new in OSPFv3.

Given this simple topology, where **R1** connects to **Core1** through a serial link, in area zero. **R1** also connects to **R2** via an ethernet segment in area twelve, we can begin analyzing the database.

**This post will cover the OSPFv3 Router LSA** <— trying to keep my posts short and structured.

#### Diagram

<a href="{{ site.url }}/assets/images/ospfv3_nmbr_one.png"><img style="border:0 initial initial;" src="{{ site.url }}/assets/images/ospfv3_nmbr_one.png" alt="" width="" height="" />

<!--more-->

### Router LSA Information

Although it has the same name, and meant for the same purpose as OSPFv2 Router LSA, there's a significant difference. The **OSPFv3** *Router LSA* does not advertise prefixes with it, only describes the Router and it's links to neighbors. Prefixes are advertised in a new LSA called **Intra-area Prefix LSA** which we'll discuss in a later post. This is a major improvement on the protocol itself, which has to do with running SPF algorithm or not.

The scalability comes of the protocol comes in to play like this; in **OSPFv2**;

  * The *Router LSA* is meant to describe the specific router as a Node in the Tree 
  * Which means, if a Router LSA is advertised, it affects the SPF tree Itself, because  
    of the implied change to the *node* itself
  * But, since prefix information is exchanged as well, even something as simple as a Subnet change  
    causes the SPF algorithm to be run. Even though that specific branch of the tree is not affected

In **OSPFv3**;

  * The *Router LSA* does the same, describes the router as a Node in the tree
  * However, prefix specific information is not exchanged 
      * therefore, subnet re-addressing does not cause a run of the SPF algorithm since this is not advertised
  * Only changes that affect the Tree itself are advertised 
      * a neighbor going down
      * an interface being removed leading to another Node (Neighbor)

### R1's Information

What better way to look at this than with good old **Core1**, as advertised by **R1**, so lets get started. Let's first look at **Core1's** *Router LSA(s)* for **Area 0**. 

<pre lang="plain">Core1#sh ipv6 ospf data router adv-router 10.0.0.1  

            OSPFv3 Router with ID (10.0.0.100) (Process ID 1)

                Router Link States (Area 0)

  LS age: 68
  Options: (V6-Bit, E-Bit, R-bit, DC-Bit)
  LS Type: Router Links
  Link State ID: 0
  Advertising Router: 10.0.0.1
  LS Seq Number: 80000090
  Checksum: 0x3850
  Length: 56
  Number of Links: 2

    Link connected to: a Transit Network
      Link Metric: 1
      Local Interface ID: 3
      Neighbor (DR) Interface ID: 3
      Neighbor (DR) Router ID: 10.0.0.2

    Link connected to: another Router (point-to-point)
      Link Metric: 64
      Local Interface ID: 5
      Neighbor Interface ID: 5
      Neighbor Router ID: 10.0.0.100
</pre>

If you compare this to the **OSPFv2** counterpart, you'll notice the difference;

<pre lang="plain">Core1#sh ip ospf data router adv-router 10.0.0.1  

            OSPF Router with ID (10.0.0.100) (Process ID 1)

                Router Link States (Area 0)

  LS age: 57
  Options: (No TOS-capability, DC)
  LS Type: Router Links
  Link State ID: 10.0.0.1
  Advertising Router: 10.0.0.1
  LS Seq Number: 80000003
  Checksum: 0x4A7B
  Length: 72
  Number of Links: 4

    Link connected to: a Stub Network
     (Link ID) Network/subnet number: 10.0.0.1
     (Link Data) Network Mask: 255.255.255.255
      Number of MTID metrics: 0
       TOS 0 Metrics: 1

    Link connected to: a Stub Network
     (Link ID) Network/subnet number: 10.1.12.0
     (Link Data) Network Mask: 255.255.255.0
      Number of MTID metrics: 0
       TOS 0 Metrics: 1

    Link connected to: another Router (point-to-point)
     (Link ID) Neighboring Router ID: 10.0.0.100
     (Link Data) Router Interface address: 10.0.1.1
      Number of MTID metrics: 0
       TOS 0 Metrics: 64

    Link connected to: a Stub Network
     (Link ID) Network/subnet number: 10.0.1.0
     (Link Data) Network Mask: 255.255.255.252
      Number of MTID metrics: 0
       TOS 0 Metrics: 64
</pre>

See how there's much more information, including *prefix information* for the area, and not just the Router and it's neighbors.

### R2's IPv6 OSPF Router LSA

If we look at **R2**, we'll notice that the information is the same from **R1** as the LSA is Area in scope. 

<pre lang="plain">R2#sh ipv6 ospf data router adv 10.0.0.1

            OSPFv3 Router with ID (10.0.0.2) (Process ID 1)

                Router Link States (Area 0)

  LS age: 369
  Options: (V6-Bit, E-Bit, R-bit, DC-Bit)
  LS Type: Router Links
  Link State ID: 0
  Advertising Router: 10.0.0.1
  LS Seq Number: 80000090
  Checksum: 0x3850
  Length: 56
  Number of Links: 2

    Link connected to: a Transit Network
      Link Metric: 1
      Local Interface ID: 3
      Neighbor (DR) Interface ID: 3
      Neighbor (DR) Router ID: 10.0.0.2

    Link connected to: another Router (point-to-point)
      Link Metric: 64
      Local Interface ID: 5
      Neighbor Interface ID: 5
      Neighbor Router ID: 10.0.0.100
</pre>

### Summary {#summary}

As you can probably see by now, this little but significant change makes **OSPFv3** a bit more efficient. This limits scope of SPF runs on the routers (not that this is a huge deal anymore). But can also affect convergence speed and unnecessary flooding of simple changes. There's <a href="http://tools.ietf.org/search/rfc5838" target="blank">work</a> to extend **OSPFv3** to support multiple address-families, such as **IPv4** and hopefully it's implemented soon. I do hear Cisco's working on this, but cannot be completely sure.