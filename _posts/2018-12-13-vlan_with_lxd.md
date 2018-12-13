---
layout: single
title:  "Configure a LXD container to be in a specific VLAN"
date:   2018-12-13 21:51:16 +0100
categories: network vlan lxd
---
I run several lxd containers on server in the basement with a trunk and multiple VLANs.
Previously, I configured an interface for each VLAN on the host.
This was very stupid and ineffective. The host was reachable from almost all the VLANs and it caused routing problems.

I looked for a solution and didn't really find anything documented.
Then I tried this and it worked !
{% highlight bash %}
sudo lxc profile device set lanprofile eth0 vlan 100
{% endhighlight %}
You can actually configure a profile with eth0 in a specific VLAN (lanprofile is the name of one of my LXD profiles).
I removed the VLAN interface on the host and it works. All my "lanprofile" containers are now in VLAN 100 (tagged).
