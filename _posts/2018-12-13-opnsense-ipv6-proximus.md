---
layout: single
title:  "IPv6 with Proximus, OPNSense and DHCPv6"
date:   2018-12-13 21:51:16 +0100
categories: network opnsense ipv6
---
I tried several time to get IPv6 connection on my [OPNSense](https://opnsense.org) with working DHCPv6.
By the way, this should work with [PFSense](https://www.pfsense.org) as well. I switched a while ago from PFSense to OPNSense because I had the feeling that PFSense was becoming too commercial.

Unfortunately, my internet provider in Belgium (Proximus) does not give fix IPv6 addresses.
So DHCPv6 is less usefull than planned.

Anyway, this is how I got it working.

First, you have to configure your WAN interface to get an IP address through DHCPv6.
![image-center](/assets/images/opnsense_ipv6/wan.png)
Then go to the bottom of the page and configure the DHCPv6 parameters.
Because Proximus does not provide any documentation, I don't know the actual size of the prefix.
I tried /56 and it did work.
![image-center](/assets/images/opnsense_ipv6/wan_dhcp6.png)
Then you need to configure your LAN interface to have an IPv6 address.
Here we use "track interface".
![image-center](/assets/images/opnsense_ipv6/lan.png)
Don't forget to configure the router advertisement.
This is how we'll tell to the clients that we use DHCPv6.
![image-center](/assets/images/opnsense_ipv6/ra.png)
Finally, configure the DHCPv6 server.
Here we configure a static second part of the IPv6 address for our clients.
Unfortunately, the first part can't be static because of Proximus.
![image-center](/assets/images/opnsense_ipv6/dhcp6.png)
