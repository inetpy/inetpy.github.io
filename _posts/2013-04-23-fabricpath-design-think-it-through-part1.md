---
title: 'Fabricpath Design, Think it Through! - part 1'
author: yandy
layout: post
permalink: /2013/04/fabricpath-design-think-it-through-part1
dsq_thread_id:
  - 1231107620
categories:
  - Networking
tags:
  - data center
  - Datacenter
  - ecmp
  - FabricPath
  - layer2
  - Nexus
  - nexus 5000
  - nexus 7000
---
## What is FabricPath

FabricPath is a Layer&#8211;2 routing architecture (yes L2 routing), using ISIS to build shortest path trees to destination bridges (using their bridge-id). Cisco offers plenty of documents <a href="http://www.cisco.com/en/US/prod/collateral/switches/ps9441/ps9670/guide_c07-690079.html" target="blank">This is one of them</a> explaining in detail what FabricPath is. FabricPath (Cisco&#8217;s implementation of a non-standard TRILL implementation) completely eliminates spanning-tree from the FP links. 

FabricPath does require F1 or F2 line-cards in a Nexus 7000 series switch. It also requires an additional license called (ENHANCED_L2) for FabricPath only. 
<!--more-->

### M Cards and FabricPath VLAN

Fabricpath is only supported on F1 or F2 modules in the Nexus 7000 switch!

Assuming F1 modules, a VLAN designated as FP VLAN cannot exist as an access or tagged VLAN on an M series of cards. Let’s say you have VLANs 100 − 150 on a 7K and you designate them as FP VLANs.

<pre lang="plain">vlan 100-150
 mode fabricpath
 exit
</pre>

This being the case, the below scenario (diagram) is not a valid design/configuration. If you try to normally trunk the any FP VLAN on an M1/2 port it will give you an error.

**This is not Valid!**  
[<img id="img" title="img" src="http://ipyandy.net/assets/images/fp_and_m_f_cards_01.jpg" alt="" width="" height="" />][img1]

What you’ll need to do is create two separate VDCs then create a Trunk from the **F-Series** card to the **M-Series** card, ultimately separately managing them. 

**This is Valid!**  
[<img id="img2" title="img2" src="http://ipyandy.net/assets/images/fp_and_m_f_cards_02.jpg" alt="" width="" height="" />][img2]

The **F2** card requires it’s own VDC for now, when mixed with **M-Series** or even **F1** line-cards in the same chassis. Therefore the original prolem is avoided, but still a cludge.

## Yandy’s Thoughts

Given the positioning of the **M-Series** vs **F-Series** modules, this design caveat is easily avaidable. However, I’ve ran into a few design meetings already, where modules were being purchesed for *port density vs features and positioning vs pricing*. This would’ve caused an issue with the requirements come deployment time.

A properly scoped **Nexus 7000** with **M-Series** modules for L3 features and **F-Series** modules for L2 features should not have this issue, but again good to keep in mind.

*If any of this has changed recently, by all means let me know, but as of now I’m not aware of any changes.*

[img1]: http://ipyandy.net/assets/images/fp_and_m_f_cards_01.jpg
[img2]: http://ipyandy.net/assets/images/fp_and_m_f_cards_01.jpg