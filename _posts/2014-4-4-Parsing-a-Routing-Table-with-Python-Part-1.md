---
title: 'Parsing a Routing Table with Python - Part 1'
author: Yandy
layout: post
image: python_banner_01.png
twitter_image: python_banner_01.png
banner_image: python_banner_01.jpeg
description: 'Using python to get contents of a routing table part 1'
summary: It's all in the API's, well, not exactly, not everything you want to get or parse is handed down in a nicely formatted XML file or JSON format. These file structures translate to some type of data structure, with relative ease. Be it a list/array a dictionary container or some sort of combination.
permalink: /2014/04/parsing-a-routing-table-with-python-part1
categories:
  - Coding
tags:
  - python
  - coding
  - networking
  - routing
  - python networking
  - python engineer
  - parsing routing table
  - parsing python
  - python parsing
  - cisco python
  - arista python
---
![](http://ipyandy.net/images/python_banner_01.jpeg)

It's all in the API's, well, not exactly, not everything you want to get or parse is handed down in a nicely formatted XML file or JSON format. These file structures translate to some type of data structure, with relative ease. Be it a list/array a dictionary container or some sort of combination.

<a href="http://www.arista.com/en" target="_blank">Arista Networks</a> has a robust API for their switches called eAPI. I wrote an intro article to eAPI not that long ago.I won’t speak for others yet, onePK from Cisco just went GA recently and still hashing it out.

But even eAPI’s robustness does not have all the bells and whistles wrapped up and ready to serve for you. While Arista engineers and developers are working hard to get them out, there’s still quite a few things that are untranslated and packed into neat data structures.

<!--more-->

### Running it

One of them being the famous show ip route [vrf (vrf_name)] command. Running this through a JSONRPC call into a switch returns a single string with all the information you’re used to seeing.

#### Something like this
[<img id="eapi01" title="arista_001.png" alt="arista_001.png" src="{{ site.url }}/assets/images/arista_001.png" />][img_1]

It isn’t very pretty or readable, however if a command is supported in json format, the output is much better in programming terms. There are indexes and keys that lead directly to information.

##### Such as
[<img id="arista_002.png" title="arista_002.png" alt="arista_002.png" src="{{ site.url }}/assets/images/arista_002.png" />][img_2]

The information is return as follows:
* a list with one index [0]
* this index contains a dictionary {interfaceStatus}
* which in turn contains many dictionarie of different values
* one example
	* key ['Management1'] is in itself a dictionary
	* contains keys to vlanInformation, interfaceMode and so on.

When coding with this type of structure, as complicated as it may seem, it’s easy to access and manipulate.

**Disclaimer:** code shown in this post is partial, and only shows the parts relevant to the information being shown. Links to full source will be placed at the end of the series of articles.

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<!-- banner_728x90-txt-disp -->
<ins class="adsbygoogle"
     style="display:inline-block;width:728px;height:90px"
     data-ad-client="ca-pub-2031545302097188"
     data-ad-slot="8561880698"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>

Lets get that output to show in the terminal prompt with a simple class method that just gets the values returned by “show interfaces brief” command.

{% highlight bash %}
# function returns a dictionary of the interfaces and their status
# along with other relevant information
def getInterfacesStatus(self):
    response = self._runCmd(['show interfaces status'])[0]['interfaceStatuses']
 
    return response
{% endhighlight %}

The above code returns the response given by the _runCMD funtion. The following script utilizes that code to print out a representation of that in a terminal.

{% highlight bash %}
$ ./scripts/netCalls.py -m arista -n sw1 -i veos-01 -f getInterfacesStatus
 
# --------------------------------------------------------------------------------
{ u'Ethernet1': { u'autoNegotigateActive': False,
                  u'bandwidth': 10000000000,
                  u'description': u'connects to vEOS-02 -> {Eth1}',
                  u'duplex': u'duplexFull',
                  u'interfaceType': u'EbraTestPhyPort',
                  u'linkStatus': u'connected',
                  u'vlanInformation': { u'interfaceForwardingModel': u'dataLink',
                                        u'vlanExplanation': u'in Po1'}},
  u'Ethernet2': { u'autoNegotigateActive': False,
                  u'bandwidth': 10000000000,
                  u'description': u'connects to vEOS-02 -> {Eth2}',
                  u'duplex': u'duplexFull',
                  u'interfaceType': u'EbraTestPhyPort',
                  u'linkStatus': u'connected',
                  u'vlanInformation': { u'interfaceForwardingModel': u'dataLink',
                                        u'vlanExplanation': u'in Po1'}},
  u'Ethernet3': { u'autoNegotigateActive': False,
                  u'bandwidth': 10000000000,
                  u'description': u'',
                  u'duplex': u'duplexFull',
                  u'interfaceType': u'EbraTestPhyPort',
                  u'linkStatus': u'connected',
                  u'vlanInformation': { u'interfaceForwardingModel': u'routed',
                                        u'interfaceMode': u'routed'}},
{% endhighlight %}

This is much nicer than our first screenshot, where the output was one long string. Theres a real data structure, there are dictionary keys, pointing to other values, explained in above.

Here’s our first screenshot in a terminal window, through the script.

{% highlight bash %}
$ ./scripts/netCalls.py -m arista -n sw1 -i veos-01 -f getRoutes
 
Codes: C - connected, S - static, K - kernel,
       O - OSPF, IA - OSPF inter area, E1 - OSPF external type 1,
       E2 - OSPF external type 2, N1 - OSPF NSSA external type 1,
       N2 - OSPF NSSA external type2, B I - iBGP, B E - eBGP,
       R - RIP, I - ISIS, A B - BGP Aggregate, A O - OSPF Summary,
       NG - Nexthop Group Static Route
 
Gateway of last resort is not set
 
 O      1.0.0.1/32 [110/11] via 1.1.3.1, Ethernet3
 O      1.0.0.2/32 [110/11] via 1.2.3.2, Ethernet4
 C      1.0.0.3/32 is directly connected, Loopback0
 O      1.1.2.0/30 [110/50] via 1.1.3.1, Ethernet3
                            via 1.2.3.2, Ethernet4
 O      1.1.2.4/30 [110/50] via 1.1.3.1, Ethernet3
                            via 1.2.3.2, Ethernet4
 C      1.1.3.0/24 is directly connected, Ethernet3
 O      1.1.4.0/24 [110/20] via 1.3.4.4, Vlan4094
 C      1.1.100.0/24 is directly connected, Vlan100
 C      1.2.3.0/24 is directly connected, Ethernet4
 O      1.2.4.0/24 [110/50] via 1.2.3.2, Ethernet4
 C      1.3.4.0/24 is directly connected, Vlan4094
 C      10.17.33.0/24 is directly connected, Ethernet9
{% endhighlight %}

While the output is “nicer” than in the screenshot, it’s the same view you get on a router. By view, I don’t mean just output to screen, I mean the actual data representation. Might as well login to the router/switch and runt he command yourself. For anything useful in coding, imaging having to parse through this string, everytime you want to find information. There’s no separation of data, it’s just one looooongggg string ;), or unicode representation.

Can we have routes or basically anything else be represented likd the Interface Status? Sure we can, I’m using the “show ip route” command, because we probably run that on a daily basis, multiple time, it’s familiar and well understood.

### The Details

What’s in a show ip route output? At least the relevant parts? For the most part, these should be good enough.

1. The protocol received from
	1. OSPF, BGP, EIGRP, Connected and so on
2. The route/prefix itself
3. The next hop ip
4. The next hop interface

### Representing the Routes

How do we represent this? What data is best suited for this? Well best is of course, as always “it depends”. The way I decided to structure it is like so.

* A dictionary of dictionaries, where the first key is the Protocol itself, O, B, I, C, S and so on.
* Those key’s values are themselves dictionaries, with keys being the actual route/prefix
* Those dictionaries are holding keys to next_hop, AD/Metric and next_hop_int.
* Those keys values are either strings or lists
	* Strings contain the AD/Metric
	* The lists contain next hop ip and next hop interfaces
	* Lists to represent the latter two, because if ECMP is in effect, there will be more than one next-hop

### Putting it Together

If you’re still not bored to death, go on to read [Part 2][2].

[img_1]: {{ site.url }}/assets/images/arista_001.png
[img_2]: {{ site.url }}/assets/images/arista_002.png
[2]: {{ site.url }}/2014/04/parsing-a-routing-table-with-python-part2