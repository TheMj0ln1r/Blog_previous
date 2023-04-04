---
layout: post
title:  "NetScan"
---

Hi guys!,

This is Mj0ln1r and i am back with a new post but this time a project devoleped with python.

I am very much intrested to do automating tasks using python. To learn python upto the advanced level i decided to make basic projects with the python.This is my first step towards the path i choosen.

I wrote this script called `NetScan` in python. I learned `basics of socket programming` , little bit about `platform` module in python and basic tools `ping`, `arp` which are useful in computer networking.

So, here's a small intro about my basic netscan script.

# NetScan

This is a basic computer networking based project the script i developed using python which can perform following tasks.

1. Get any websites IP
2. Discover live hosts
3. Discover Hosts and MAC

I used ping to send the ICMP packets to every device connected to network, the response from that device has been analyzed and printed the devices which are responded for the ICMP request.
This process could be slow if the number of hosts to be scanned are increased, but it will works fine as it uses just a basic ping which is available in any operating system.

A TCP scan is also used to get the active hosts in a network this will overcome the ping scan drawbacks, i.e the ICMP packets can be blocked by a firewall in this process we may get the host status as unreachable. To overcome this are going to sent tcp packets to the port 135 using socket module in python.

An arp scan is used to the MAC adrresses of all the hosts connected in a network.

I devoloped this script with less additional modules which can make the scan slow , but this can be executed in any system without any additional requirements.

### Requirements

You may require some python modules

1. subprocess
2. socket
3. platform

These modules will be pre installed on every system, if not then install them with `pip3 or pip`
### Installation

	git clone https://github.com/TheMj0ln1r/NetScan.git
	cd NetScan
	python3 NetScan.py

### Preview 
![Preview](/assets/img/project_img/netscan.png){: w="600",h="600"}

> If anything should be modified in the script please let me know.
{: .prompt-info }

You can find complete source code of NetScan here : [https://github.com/TheMj0ln1r/NetScan](https://github.com/TheMj0ln1r/NetScan)