---
title: EIGRP Route Filtering with NX-OS
author: yandy
layout: post
permalink: /2011/07/eigrp-route-filtering-with-nx-os
dsq_thread_id:
  - 1235490767
categories:
  - Networking
tags:
  - cisco nexus
  - eigrp
  - Nexus
  - nexus 7000
  - nexus 7010
  - nx-os
  - route filtering
---
### Overview

So we all know how to filter EIGRP routes; right? create a prefix-list or access-list then apply that to the routing process. You can also do **offset-lists**, all that good stuff. This is where the good part comes in; and where the frustration sometimes sets in as well. I really wish BUs within Cisco will talk more, and standardize the way things are done. There’s the whole summary routes are done under the interface in IOS, so is authentication. Then filtering is done under the routing process and so forth. Again, this is for vanilla IOS, then comes NX-OS, which at the CLI level looks allot like IOS.
<!--more-->

Before going into detail, let’s just see how this is done in IOS for those that may not know or remember.

### IOS EIGRP Filtering - Basics

So here’s the diagram of **R1** and **R2** connected via an ethernet link, and using EIGRP to pass routes. They each have a loopback interface with IP address of 10.0.0.1/32 and 10.0.0.2/32 respectively.

[<img id="img1" title="img1" src="http://ipyandy.net/assets/images/EIGRP_R1_R2.png" alt="" width="" height="" />][img1]

#### R1 Configuration {#r1configuration}

Here’s the configuration as it stands, before any filtering for **R1**.

<pre lang="plain">R1(config-router)#do sh run | sec router eigrp
router eigrp 100
 network 10.0.0.0
 network 10.1.2.0 0.0.0.255
 no auto-summary
 passive-interface default
 no passive-interface Ethernet0/3
</pre>

Here’s **R1’s** EIGRP routing table before any filtering is applied.

<pre lang="plain">R1(config-router)#do sh ip route eigrp

      10.0.0.0/8 is variably subnetted, 8 subnets, 3 masks
D        10.0.0.2/32 [90/409600] via 10.1.2.2, 00:00:02, Ethernet0/3
D        10.17.2.0/30 [90/2195456] via 10.1.2.2, 00:04:18, Ethernet0/3
</pre>

We can see that 10.0.0.2/32 is learned via Ethernet0/3 with a next hop of 10.1.2.2 which is **R2’s** connected segment on Ethernet0/3. There’s also the 10.17.2.0/30 network, but we’re not worried about filtering that, it’s a connection to another router not shown.

#### Apply some Filtering

Now let’s have a little fun and apply some filtering;

Here’s our options:

<pre lang="plain">R1(config)#router eigrp 100
R1(config-router)#no auto
R1(config-router)#distribute-list ?
  &lt;1-199>      IP access list number
  &lt;1300-2699>  IP expanded access list number
  WORD         Access-list name
  gateway      Filtering incoming address updates based on gateway
  prefix       Filter prefixes in address updates
  route-map    Filter prefixes based on the route-map

R1(config-router)#offset-list ?
  &lt;1-99>       Access list of networks to apply offset (standard range)
  &lt;1300-1999>  Access list of networks to apply offset (extended range)
  WORD         Access-list name
</pre>

We can use a distribute-list and apply a route-map, prefix-list or access-list. There’s also an offset-list to set high enough metrics where that route wont show up in the routing table. There’s also (not shown) the distance command which can be used to filter or manipulate routes.

We’re going to use the first method and just apply a **prefix-list** to a **distribute-list** under the EIGRP process.

Like so:

<pre lang="plain">ip prefix-list NO-R2-LOOPBACK-EIGRP seq 5 deny 10.0.0.2/32
ip prefix-list NO-R2-LOOPBACK-EIGRP seq 10 permit 0.0.0.0/0 le 32

R1(config)#do sh run | sec router eigrp
router eigrp 100
 distribute-list prefix NO-R2-LOOPBACK-EIGRP in Ethernet0/3
 network 10.0.0.0
 network 10.1.2.0 0.0.0.255
 no auto-summary
 passive-interface default
 no passive-interface Ethernet0/3
</pre>

Now let’s take a look and see what this did to our routing table:

<pre lang="plain">R1(config)#do sh ip route eigrp

Gateway of last resort is not set

      10.0.0.0/8 is variably subnetted, 8 subnets, 3 masks
D        10.17.2.0/30 [90/2195456] via 10.1.2.2, 00:04:18, Ethernet0/3
</pre>

And it works, we’re only seeing that other network and the loopback address is gone from the table. Now why all this to cover NX-OS and EIGRP. Because it’s good to understand the basics, and it’s likely that you’ll see this, before you ever touch an NX-OS device, such as a Nexus 5548 or 5596, or the Nexus 7000 series.

### NX-OS EIGRP Route Filtering

And now for the fun part, filtering with **NX-OS** on the Cisco Nexus 7010 Data Center switch. There’s allot of similarities in command line structure, but there’s also allot of subtle differences. Just enough to sometimes give you some kind of headache. 

#### Basic Configs

Here’s our very basic **router eigrp** configuration for NX-OS.

<pre lang="plain">router eigrp 100
  redistribute direct route-map DIRECT-TO-EIGRP
  redistribute static route-map STATIC-TO-EIGRP
  authentication mode md5
  authentication key-chain EIGRP-KEY
</pre>

Don’t worry too much about the route-maps in the configuration, they’re there, because they’re needed for something else unrelated. Which by the names, I’m guessing you can take a stab at what they’re for.

Here are our options under **router eigrp** for anything we want to do here.

<pre lang="plain">N7K-01(config-if)# router eigrp 100
N7K-01(config-router)# ?
  address-family         Configure an address-family
  authentication         Configures EIGRP authentication subcommands
  autonomous-system      Specify AS number for Address Family
  default-information    Control origination of a default route
  default-metric         Set metric of redistributed routes
  distance               Define an administrative distance
  flush-routes           Flush routes in RIB during restart
  graceful-restart       Peer resync without adjancency reset
  log-adjacency-changes  Log changes in adjacency state
  log-neighbor-warnings  Enable/Disable IP-EIGRP neighbor warnings
  maximum-paths          Forward packets over multiple paths
  metric                 Modify EIGRP routing metrics and parameters
  no                     Negate a command or set its defaults
  redistribute           Redistribute information from another routing protocol
  router-id              Router-id for this EIGRP process
  shutdown               Shutdown this instance of EIGRP
  stub                   Set IP-EIGRP as stubbed router
  timers                 Set EIGRP timers
  vrf                    Configure VRF information
  end                    Go to exec mode
  exit                   Exit from command interpreter
  pop                    Pop mode from stack or restore from name
  push                   Push current mode to stack or save it under name
  where                  Shows the cli context you are in


N7K-01(config-router)# distribute-list?
                                    ^
% Invalid command at '^' marker.
N7K-01(config-router)# distribute-list

N7K-01(config-router)# offset-list?
                                 ^
% Invalid command at '^' marker.
N7K-01(config-router)# offset-list
</pre>

As you can see, there’s no **distribute-list** command or **offset-list** option either. There is the distance command, which again goes with Cisco’s consistent inconsistencies :). So where are those options if you want to filter anything under NX-OS? They’ll be under each individual interface.

Like so:

<pre lang="plain">interface Vlan50
  ip address 10.1.100.1 255.255.255.0
  ip router eigrp 100
  ip authentication mode eigrp 100 md5
  ip authentication key-chain eigrp 100 EIGRP-KEY
  ip distribute-list eigrp 100 prefix-list NO-DEFAULT-VPN in

N7K-01(config-router)# int vlan 50
N7K-01(config-if)# ip offset-list ?
  eigrp  EIGRP interface configuration commands
</pre>

You can see there’s already a **distribute** list applied with the **ip distribute-list eigrp <as_#>** interface configuration command. There’s an option as well to apply an access-list, although we’re using a prefix-list; I like prefix-list where possible, much cleaner intention in my opinion. And, there’s also the offset-list command under the same interface configuration, if you wanted to do some metric offsets.

### Summary

There’s no doubt that the Nexus line of switches are a fun product to play with. But so is any other network equipment, when it’s not broken. However, there’s always discrepancies it seems in the Cisco product line on how to apply certain commands. Not always where you expect them to be, where you’re thought they would be. Yes, that’s what the command reference and configuration guides are for. But sometimes, just sometimes, you’ll want to have some kind of consistency in things; don’t ya? Hopefully this helps someone out :)

[img1]: http://ipyandy.net/assets/images/EIGRP_R1_R2.png