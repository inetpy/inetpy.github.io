---
title: 'Parsing a Routing Table with Python - Part 2'
author: Yandy
layout: post
banner_image: python_banner_02.png
permalink: /2014/04/parsing-a-routing-table-with-python-part2
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
### GETTING READY

In the [previous article][1] (they’re split to make them easier to read) I talked about the theory and representation. How a properly formatted data structure and a non-formatted data structure look like. Here, we’re going to convert the output of the popular show ip route command into a “proper” data structure, and not just a string.

<!--more-->

### (./python) THE CODE

Again, any code “snippets” are not the full representation of the program. The full code is available on <a href="http://ipyandy.net/pnil/" target="_blank">github under my PNIL repo</a> and can be cloned, downloaded or forked.

##### Running this line

{% highlight bash %}
$ ./scripts/netCalls.py -m arista -n sw1 -d veos-01 -f getRoutes
{% endhighlight %}

Which runs from main in the containing netCalls.py, follows these steps.
1. -m, –manufacturer [arista|cisco|juniper] and so on
2. only arista implemented so far
3. -n, –name [trivial name for the device] this is optional
4. -i, –ip_address or -d –dns_name [ip_address] or [dns_name]
5. these are redundant and can take the same input
6. -f, –function [getRoutes|getSystemMac] and more
7. 
There are other options to the script, but that’s for the documentation to cover, when I get around to it.

So what does it do? By some magic code into the script, the getRoutes function in **eapi.py** gets called.

{% highlight python %}
def main():
    if ARISTA and ARGUMENTS:
        args = initArgs()
        function = args['function'] if args['function'] else None
 
        sw1 = build(args)
        result = sw1.run(function, args)
 
        printResult(result)
        # REST OF [OUTPUT TRUNCATED]
{% endhighlight %}

* First we build the command line arguments
* args = initArgs()
* This calls a utility function that parses the terminal for the options passed int
* We then setup the “function” to be called, by seeing if it was passed with the -f or –function flag
* The device (sw1) is built by calling the build(args) function
* This parses the arguments further to initialize other options
* IP address
* device name
* even username or password if passed in with (-u and -p), otherwise defaults are set
* using defaults is not recommend
* result is then assigned the value(s) returned by running the function and options passed in sw1.run(function, args)
* The results are the printed to screen printResult(result)
* this a formatting function, to make it look pretty when outputted to screen.

##### This following code builds the devie (in this case sw1).

{% highlight python %}
def build(args):
    # sets dev_name if -n is used, otherwise generic 'dev' is used
    name = args['name'] if args['name'] else 'net1'
 
    # create device with ip_address or dns_name and manufacturer info
    _host = args['ip_address'] if args['ip_address'] else args['dns_name']
 
    net_dev = netDevice()
    net_dev.initialize(_host, args['manufacturer'], name)
 
    # if login information entered, use it, otherwise default
    if args['username'] and args['password']:
        net_dev.setLogin(args['username'], args['password'])
 
    return net_dev
{% endhighlight %}

* This names the device if a name is passed in with -n or –name, otherwise a generic name is assigned
* The host is then set wether the and IP is given or a DNS Hostname
* the device is then created by calling neetDevice()
* depending on the manufacturer flag, the correct api is assigned to net_device
* the device is then initialized by calling the initialize() method
* if optional username and passwords are assigned, this get set
* and finally the correct device is returned/assigned to the calling member in main.

When sw1.run() is called in main, the correct sequence of operations happen, based on the name of the function used, it will call in our case getRoutes() inside of the eapi library.

{% highlight python %}
def getRoutes(self, args=None):
    if args and args['vrf'] and args['options']:
        routes = self._runCmdText(['show ip route vrf {0} {1}\
            '.format(args['vrf'], args['options'])])[0]['output']
    elif args and args['vrf']:
        routes = self._runCmdText(['show ip route vrf {0}\
            '.format(args['vrf'])])[0]['output']
    elif args and args['options']:
        routes = self._runCmdText(['show ip route {0}\
            '.format(args['options'])])[0]['output']
    else:
        routes = self._runCmdText(['show ip route'])[0]['output']
 
    return standardRoutes.getRoutes(routes)
{% endhighlight %}

Based on the options passed in, this runs the show ip route command with the various options if any, otherwise a simple command is passed.

The result is then passed on to standardRoutes.getRoutes() library method to parse the returned string, and format it into a data structure.

##### Here’s the getRoutes() method from standardRoutes.getRoutes()

{% highlight python %}
@classmethod
def getRoutes(cls, routes):
    routes_list = cls.createRoutesList(routes)
 
    p_keys = cls.getRoutesProtocol(routes_list)
    routes_dict = {key: {} for key in p_keys}
 
    for line in routes_list:
        p_key = cls.getRoutesProtocol([line])
        prefix = cls.getRoutePrefixes([line])
 
        # if p_match and pr_match:
        if p_key and prefix:
            ad_metric = cls.getADMetric([line])
            next_hop = cls.getNextHop([line])
            next_hop_int = cls.getNextHopInterface([line])
            routes_dict[p_key[0]][prefix[0]] = {'ad_metric': ad_metric[0],
                                                    'next_hop': next_hop,
                                                    'next_hop_int': next_hop_int
                                        }
 
    return routes_dict
{% endhighlight %}

This method calls a getRoutesList()

{% highlight python %}
routes_list = cls.createRoutesList(routes)
{% endhighlight %}

Which if you guessed, returns a list of the string given. This makes it easier to run line by line and find the relevant information. I won’t show the code to keep confusion as little as possible, but getRoutesList splits the output of “show ip route” at each new line, and creates a list of those lines.

We then create a dictionary of the protocols (OSPF, BGP, Static..) contained in the list.

{% highlight python %}
p_keys = cls.getRoutesProtocol(routes_list)
routes_dict = {key: {} for key in p_keys}
{% endhighlight %}

The next few lines are the core of the script, where we parse the output line by line and append the relevant information to our dictionary.

{% highlight python %}
for line in routes_list:
        p_key = cls.getRoutesProtocol([line])
        prefix = cls.getRoutePrefixes([line])
 
        # if p_match and pr_match:
        if p_key and prefix:
            ad_metric = cls.getADMetric([line])
            next_hop = cls.getNextHop([line])
            next_hop_int = cls.getNextHopInterface([line])
            routes_dict[p_key[0]][prefix[0]] = {'ad_metric': ad_metric[0],
                                                    'next_hop': next_hop,
                                                    'next_hop_int': next_hop_int
                                        }
{% endhighlight %}

The Protocols and Prefixes keys are parsed, and if they exist the corresponding ad_metric, next_hop and net_hop_int are assigned. Finally the dictionary is returned (assigned) to the calling function or variable. Main then passes that information to a printing method and outputs the contents to screen.

The utility functions, such as getRoutePrefixes() use a regular expression to parse the text.

{% highlight python %}
# RegularExpression to parse the line of text and find the IP address.
PREFIX_RE = re.compile(r'(([\d]{1,3}\.){3}([\d]{1,3}){1}((/[\d]{1,2})?)((?=\s?\[)|(?=\sis)))', re.IGNORECASE)
@classmethod
def getRoutePrefixes(cls, search_list):
    prefixes = []
    for p in search_list:
        # The test for protocol first in the prefix line is necessary
        # the regEX matching the prefix, sometimes matches an un-necessary line
        # such as 10.0.0.0/8 is variably subneted, under this line, are the actual prefixes
        protocol = cls.getRoutesProtocol([p])
        if protocol:
            pr_match = PREFIX_RE.search(p)
            if pr_match:
                prefixes.append(pr_match.group(0))
 
    return prefixes
{% endhighlight %}

The others work much in the same, except with different regular expressions to find the indidual values we’re interested in.

##### This is finally the pretty looking output to screen.

{% highlight bash %}
# --------------------------------------------------------------------------------
# ARISTA DATASTRUCTURE REPRESENTATION
# --------------------------------------------------------------------------------
 
{ u'C': { u'1.0.0.3/32': { 'ad_metric': '0/0',
                           'next_hop': [u'connected'],
                           'next_hop_int': [ u'Loopback0']},
          u'1.1.100.0/24': { 'ad_metric': '0/0',
                             'next_hop': [u'connected'],
                             'next_hop_int': [ u'Vlan100']},
          u'1.1.3.0/24': { 'ad_metric': '0/0',
                           'next_hop': [u'connected'],
                           'next_hop_int': [ u'Ethernet3']},
          u'1.2.3.0/24': { 'ad_metric': '0/0',
                           'next_hop': [u'connected'],
                           'next_hop_int': [ u'Ethernet4']},
          u'1.3.4.0/24': { 'ad_metric': '0/0',
                           'next_hop': [u'connected'],
                           'next_hop_int': [u'Vlan4094']},
          u'10.17.33.0/24': { 'ad_metric': '0/0',
                              'next_hop': [u'connected'],
                              'next_hop_int': [ u'Ethernet9']}},
  u'O': { u'1.0.0.1/32': { 'ad_metric': u'110/11',
                           'next_hop': [u'1.1.3.1'],
                           'next_hop_int': [ u'Ethernet3']},
          u'1.0.0.2/32': { 'ad_metric': u'110/11',
                           'next_hop': [u'1.2.3.2'],
                           'next_hop_int': [ u'Ethernet4']},
          u'1.1.2.0/30': { 'ad_metric': u'110/50',
                           'next_hop': [ u'1.1.3.1',
                                         u'1.2.3.2'],
                           'next_hop_int': [ u'Ethernet3',
                                             u'Ethernet4']},
          u'1.1.2.4/30': { 'ad_metric': u'110/50',
                           'next_hop': [ u'1.1.3.1',
                                         u'1.2.3.2'],
                           'next_hop_int': [ u'Ethernet3',
                                             u'Ethernet4']},
          u'1.1.4.0/24': { 'ad_metric': u'110/20',
                           'next_hop': [u'1.3.4.4'],
                           'next_hop_int': [u'Vlan4094']},
          u'1.2.4.0/24': { 'ad_metric': u'110/50',
                           'next_hop': [u'1.2.3.2'],
                           'next_hop_int': [ u'Ethernet4']}}}
 
# --------------------------------------------------------------------------------
{% endhighlight %}

The output (how it looks) itself is not all the important, it’s the way the data is structured. We can find a specific route within OSPF and see the details. This structure can also be passed into a CSV file or database. The possabilities are endless and having a proper data structure makes it all much easier. Now I’m sure the APIs will be updated to include this, but it was a fun exercise none the less.

### BONUS

The RegEx that do most of the work behind the scenes from the calling methods. Still working on finishing up the list, as I stated, only Arista is implemented somewhat fully.

{% highlight python %}
# FOR IOS / ARISTA
PROTOCOL_RE = re.compile(r'(?&lt;=\s)(([\w])|([\w]\s[\w]+[\d]?)|([\w]+?\*))(?=\s+[\d]+\.)')
PREFIX_RE = re.compile(r'(([\d]{1,3}\.){3}([\d]{1,3}){1}((/[\d]{1,2})?)((?=\s?\[)|(?=\sis)))')
# -----------------
 
# FOR NX-OS DEVICES
PREFIX_NX_RE = re.compile(r'([\d]{1,3}\.){3}([\d]){1,3}/[\d]{1,2}(?=,)')
# -----------------
 
# AD_METRIC WORKS ON IOS/ARISTA AND NX-OS
AD_METRIC_RE = re.compile(r'(?&lt;=\[)([0-9]{1,3}/[0-9]{1,3})(?=\])')
# ---------------------------------------
 
# NEXTHOP_IP WORKS ON IOS/ARISTA AND NX-OS
NEXTHOP_IP = re.compile(r'(((?&lt;=[\w]{3}\s)(([\d]{1,3}\.){3}[\d]{1,3})(?=,))|connected)')
NEXTHOP_INT_RE = re.compile(r'((?&lt;=\d,\s)|(?&lt;=connected,\s))(([\w])+([\d]{1,3})(/?)([\d]{1,3})?(/?)([\d]{1,3})?(/?)([\d]{1,3})?)|Null0')
# ---------------------------------------
{% endhighlight %}

We can also call the script with an -o or –options flag to see specific protocol routes.

{% highlight bash %}
$ ./scripts/netCalls.py -m arista -n sw1 -i veos-01 -f getRoutes -o ospf
 
# --------------------------------------------------------------------------------
# ARISTA DATASTRUCTURE REPRESENTATION
# --------------------------------------------------------------------------------
 
{ u'O': { u'1.0.0.1/32': { 'ad_metric': u'110/11',
                           'next_hop': [u'1.1.3.1'],
                           'next_hop_int': [ u'Ethernet3']},
          u'1.0.0.2/32': { 'ad_metric': u'110/11',
                           'next_hop': [u'1.2.3.2'],
                           'next_hop_int': [ u'Ethernet4']},
          u'1.1.2.0/30': { 'ad_metric': u'110/50',
                           'next_hop': [ u'1.1.3.1',
                                         u'1.2.3.2'],
                           'next_hop_int': [ u'Ethernet3',
                                             u'Ethernet4']},
          u'1.1.2.4/30': { 'ad_metric': u'110/50',
                           'next_hop': [ u'1.1.3.1',
                                         u'1.2.3.2'],
                           'next_hop_int': [ u'Ethernet3',
                                             u'Ethernet4']},
          u'1.1.4.0/24': { 'ad_metric': u'110/20',
                           'next_hop': [u'1.3.4.4'],
                           'next_hop_int': [u'Vlan4094']},
          u'1.2.4.0/24': { 'ad_metric': u'110/50',
                           'next_hop': [u'1.2.3.2'],
                           'next_hop_int': [ u'Ethernet4']}}}
 
# --------------------------------------------------------------------------------
{% endhighlight %}

We can even pass a –vrf flag and see those routes.

{% highlight bash %}
$ ./scripts/netCalls.py -m arista -n sw1 -i veos-01 -f getRoutes --vrf mgmt
 
# --------------------------------------------------------------------------------
# ARISTA DATASTRUCTURE REPRESENTATION
# --------------------------------------------------------------------------------
 
{ u'C': { u'192.168.31.0/24': { 'ad_metric': '0/0',
                                'next_hop': [ u'connected'],
                                'next_hop_int': [ u'Management1']}},
  u'S': { u'0.0.0.0/0': { 'ad_metric': u'1/0',
                          'next_hop': [u'192.168.31.1'],
                          'next_hop_int': [ u'Management1']}}}
 
# --------------------------------------------------------------------------------
{% endhighlight %}

Here’s combining –vrf and the -o flags to find the connected routes inside a vrf (connected, because it’s only a managment vrf and has no protocol routes).

{% highlight bash %}
$ ./scripts/netCalls.py -m arista -n sw1 -i veos-01 -f getRoutes --vrf mgmt -o connected
 
# --------------------------------------------------------------------------------
# ARISTA DATASTRUCTURE REPRESENTATION
# --------------------------------------------------------------------------------
 
{ u'C': { u'192.168.31.0/24': { 'ad_metric': '0/0',
                                'next_hop': [ u'connected'],
                                'next_hop_int': [ u'Management1']}}}
 
# --------------------------------------------------------------------------------
{% endhighlight %}

We can even pass the specific route we’re looking for with the –options, -o flag.

{% highlight bash %}
$ ./scripts/netCalls.py -m arista -n sw1 -i veos-01 -f getRoutes -o 1.1.2.0
 
# --------------------------------------------------------------------------------
# ARISTA DATASTRUCTURE REPRESENTATION
# --------------------------------------------------------------------------------
 
{ u'O': { u'1.1.2.0/30': { 'ad_metric': u'110/50',
                           'next_hop': [ u'1.1.3.1',
                                         u'1.2.3.2'],
                           'next_hop_int': [ u'Ethernet3',
                                             u'Ethernet4']}}}
 
# --------------------------------------------------------------------------------
{% endhighlight %}

### SUMMARY

The library is by no means complete, while we can pass a vrf and protocol together. Passing a vrf, protocol and specific route is not possible. Even passing two arguments to –options to find a specific OSPF route or similar is not supported. Mainly because the version of EOS I’m working on does not support calling “show ip route ospf [prefix].

I called the class standardRoutes because technically, this works with Cisco IOS or IOS-XE output as well. Working with NX-OS and XR is different but can be implemented, just wont be supported in the same class.

### BONUS++

An output of of a Cisco IOS show ip route command. This was copied manually into the getRoutes method.

{% highlight bash %}
cisco_str = '''Codes: C - connected, S - static, R - RIP, M - mobile, B - BGP
     D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area 
     N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
     E1 - OSPF external type 1, E2 - OSPF external type 2
     i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2
     ia - IS-IS inter area, * - candidate default, U - per-user static route
     o - ODR, P - periodic downloaded static route
 
      Gateway of last resort is 10.16.0.1 to network 0.0.0.0
 
      C    192.168.31.0/24 is directly connected, Vlan31
           66.0.0.0/24 is subnetted, 1 subnets
      B       66.176.87.0 [200/0] via 10.16.0.1, 1d16h
           10.0.0.0/8 is variably subnetted, 6 subnets, 3 masks
      C       10.17.31.0/24 is directly connected, Vlan31
      C       10.16.0.2/32 is directly connected, Loopback0
      C       10.16.1.0/30 is directly connected, GigabitEthernet1/48
      O       10.16.0.1/32 [110/2] via 10.16.1.1, 1w0d, GigabitEthernet1/48
      C       10.17.33.0/24 is directly connected, Vlan33
      C       10.17.37.0/24 is directly connected, Vlan37
      C    192.168.33.0/24 is directly connected, Vlan33
      B*   0.0.0.0/0 [200/0] via 10.16.0.1, 1w0d'''
{% endhighlight %}

##### Output

{% highlight bash %}
# --------------------------------------------------------------------------------
# CISCO DATASTRUCTURE REPRESENTATION
# --------------------------------------------------------------------------------
 
{ 'B': { '66.176.87.0': { 'ad_metric': '200/0',
                          'next_hop': ['10.16.0.1'],
                          'next_hop_int': ['1d16']}},
  'B*': { '0.0.0.0/0': { 'ad_metric': '200/0',
                         'next_hop': ['10.16.0.1'],
                         'next_hop_int': ['1w0']}},
  'C': { '10.16.0.2/32': { 'ad_metric': '0/0',
                           'next_hop': ['connected'],
                           'next_hop_int': ['Loopback0']},
         '10.16.1.0/30': { 'ad_metric': '0/0',
                           'next_hop': ['connected'],
                           'next_hop_int': [ 'GigabitEthernet1/48']},
         '10.17.31.0/24': { 'ad_metric': '0/0',
                            'next_hop': ['connected'],
                            'next_hop_int': ['Vlan31']},
         '10.17.33.0/24': { 'ad_metric': '0/0',
                            'next_hop': ['connected'],
                            'next_hop_int': ['Vlan33']},
         '10.17.37.0/24': { 'ad_metric': '0/0',
                            'next_hop': ['connected'],
                            'next_hop_int': ['Vlan37']},
         '192.168.31.0/24': { 'ad_metric': '0/0',
                              'next_hop': ['connected'],
                              'next_hop_int': ['Vlan31']},
         '192.168.33.0/24': { 'ad_metric': '0/0',
                              'next_hop': ['connected'],
                              'next_hop_int': ['Vlan33']}},
  'O': { '10.16.0.1/32': { 'ad_metric': '110/2',
                           'next_hop': ['10.16.1.1'],
                           'next_hop_int': ['1w0']}}}
 
# --------------------------------------------------------------------------------
{% endhighlight %}

Make sure to [Subscribe my RSS feed][2] and follwow me on Twitter [@IPyandy][3].

[1]: {{ site.url }}/2014/04/parsing-a-routing-table-with-python-part1
[2]: http://ipyandy.net/feed.xml
[3]: https://twitter.com/IPyandy