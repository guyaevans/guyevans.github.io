---
title: The road to 10 Gbps internet - Mikrotik CHR Strangeness
date: 2024-04-30T20:12:10.480Z
featuredImage: feature.jpeg
---
## How it started
My server[^Homelab] that lives in [Milywan](https://milkywan.fr)'s Croissy-Beaubourg/CBO location actually has a 10G NIC, as it was one of my requirements when setting it up. Something I have never really used its full potential. 

__Today that changed__ _(sort of)._

[^Homelab]: [I posted two articles](https://guy-evans.com/series/vps-to-a-coloed-server-my-homelab-journey/) about going from a single to virtual private server to my own server in a colocated rack.

Milkywan has a group chat on telegram where we talk about tech and other things between members. One of the guys there was looking for someone with a Mikrotik router “on-net” to run some bandwidth tests. 

— *I’m trying to find out if and why I’m limited to 1G*

Mikrotik has a built in tool for that:
> The Bandwidth Tester can be used to measure the throughput to another MikroTik router (either wired or wireless) and thereby help to discover network "bottlenecks" - [Mikrotik](https://help.mikrotik.com/docs/display/ROS/Bandwidth+Test)


Well I volunteered my CHR [^1] instance. </br></br></br>
{{<figure src="/img/jack_whatcouldgowrong.gif">}}
## A good start
So I dm’d him some connection details and he started his tests!

[^1]: Cloud Hosted Router (CHR) is a RouterOS version intended for running as a virtual machine. It supports the x86 64-bit architecture and can be used on most of the popular hypervisors such as VMWare, Hyper-V, Proxmox - [Mikrotik - Help](https://help.mikrotik.com/docs/display/ROS/Cloud+Hosted+Router%2C+CHR)

{{< figure src="./IMG_4388.jpeg" caption="Transmitting at 2.7G - not bad!">}}

Not bad for an old R620 - After some changes - my colleague ran another test ...

We managed 4G. (I forgot to take a screenshot) Ok but still not quite our the target. 

## The bottleneck

I then noticed something …

{{<figure src="IMG_4389.jpeg" caption="AH! - A bottleneck">}}

I only ever gave my CHR instance 4 vCPUs because why should I need more? But with out tests we were maxing them out. Simple solution for that problem:  Throw more vCPUs at it. I gave the VM 10 vCPUs and restarted it. 

Another test later, according to Milkywan’s weathermap [^wm] that got us to 5G but we wanted more. 

{{<figure src="IMG_4390.jpeg" caption="You can see 5Gbps coming in from the router named cer2024.edge.tls (lower middle)">}} 

[^wm]: A Weathermap is a map that displays the carriers network with overlays displaying statistics

## The ban and an oddity

While discussing some more ideas we left the test running until we got a message in the group chat:

— *Hey guys, your triggering alerts on our monitoring systems* 🤣 
_-One of the admins_

Well yes 5Gbps constant traffic will make a NOC at a small ISP look at whats happening. 

So I decided to throw it a another socket of 12 vCPUs for a total of 24 vCPUs thinking **more cpu = more bandwidth.**
{{<figure src="/img/jeremy-clarkson-sometimes-my-genius.gif">}}

That's where I found and oddity with CHR, it won’t recognise a second CPU socket. So another restart later we had a single socket cpu with 24 vCPUs. Which got us to 6G. 

While my colleague was running some other tests in parallel we got another message:

— *Well you’ve just triggered the automatic DDoS mitigation*[^anti-ddos] 😅

[^anti-ddos]: **DDoS mitigation** is a set of [network management](https://en.wikipedia.org/wiki/Network_management "Network management") techniques and/or tools, for resisting or mitigating the impact of [distributed denial-of-service attack](https://en.wikipedia.org/wiki/Distributed_denial-of-service_attack "Distributed denial-of-service attack")

That locked us out for 15 mins. But after that we ran another test and got … 6 G … again. 

We also found out after the fact that we triggering the ddos mitigation on another of Milkywan’s routers. 

We stopped after because we could continue optimising all night, trigger more DDOS mitigation and get nowhere further.

I’m still happy though, my little R620 in the colo can pull decent traffic from the internet. I’ll be waiting for my next opportunity to chase the missing 4Gb. 
