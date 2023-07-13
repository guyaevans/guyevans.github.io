---
title: 802.1x Port Authentication with ARUBA switches
date: 2023-07-13T12:34:34.427Z
featuredImage: thomas-jensen-ISG-rUel0Uw-unsplash.jpg
summary: "Imagine a situation where you have a public meeting room equipped with accessible Ethernet ports, and you want to ensure secure access to those ports. One of the options is 802.1X Port Authentication - Lets take a look at the configuration on the switch side"
tags:
  - 802.1X
  - Radius
  - Security
  - Switch
  - Aruba
  - Vlan
---
## 802.1x - Why?

In the enterprise, we often use 802.1x to secure access to wired and wireless networks. For wired networks 802.1x allows us to require authentication at the port level on a supported switch which also allows us to dynamically assign VLANs to the port or connecting device.

Imagine a situation where you have a public meeting room equipped with accessible Ethernet ports, and you want to ensure secure access to those ports. In such a scenario, you have two options to consider:

1. The first option is to disable any unused ports. While this approach provides some level of security, it comes with a drawback. What if someone needs to use the port? In that case, you would have to manually log in to the switch, enable the port temporarily, and then disable it again once it's no longer in use. This process can be quite cumbersome and timewaster for IT staff.

1. The second and more efficient option is to implement 802.1X. This standard offers a superior solution by keeping the ports active at all times. With 802.1X, when a company device is plugged into the port, it will automatically gain access to the company network on the assigned VLAN. However, if an unknown device is connected, it will be assigned to a VLAN of your choice, typically a public network with only internet access. This segregation ensures that unauthorized devices cannot access the company network. The best part is that you no longer need to manually enable and disable ports whenever access is required.

## What do we need?

* A RADIUS server (at IP 192.168.10.2) - we are using Microsoft NPS (I won't be detailing the configuration of NPS here but may do so in the futur)
* A Managed Switch - Could be any any managed switch which can do port authentication, I'm doing this on an [ARUBA 2930F](https://www.arubanetworks.com/products/switches/access/2930f-series/) (which is a nice switch by the way, if only it could make it's way home from work :wink: wink: )
* Some VLANs
  * Company VLAN: 10
  * Public VLAN: 20
* A Windows computer

## Let's get configing
### Switch Configuration
#### VLAN Setup

First thing we need to do is add the vlans if not already done. I've tagged the vlans on all the ports for simplicity sake.

```
vlan 10 name VLAN10_Company
vlan 10 tagged all
vlan 20 name VLAN20_Public
vlan 20 tagged all
vlan 30 name VLAN30_Phones
vlan 30 tagged all
```

#### Radius Server and Authentication 
Next we need to configure the remote RADIUS server (our NPS server) and tell the switch the 'secretkey'

```
radius-server host 192.168.10.2 key "secretkey"
```

Then we configure EAP-RADIUS, this enables the switch to forward the authentication packets from the clients to the Radius server.

```aaa authentication port-access eap-radius```

We then enable 802.1x on our switch ports, here we are doing it on a a range of ports but you can specify individual ports, or separate them with a comma. We tell the switch to put autorised clients on vlan 10 (auth-vid) and non autorised clients on vlan 20 (unauth-vid)

```
aaa aaa port-access authenticator 1-10
aaa aaa port-access authenticator 1-10 unauth-vid 20
aaa aaa port-access authenticator 1-10 auth-vid 10
aaa aaa port-access authenticator active
```

With that our switch configuration is done. Or is it?

#### (Bonus) What about IP Phones?

You may have noticed I included a vlan for phones, as this was a huge question I had on my first roll out of 802.1x. Will I need to add each phone to our Radius server and configure each phone for the authentication or use MAB[^MAB]?

[^MAB]: MAC Authentication Bypass - [MAB - networklessons.com](https://networklessons.com/cisco/ccie-routing-switching-written/mac-authentication-bypass-mab)

No! Instead we will make the vlan a voice vlan with the following command which will allow the phone to use LLDP to fetch its assigned vlan bypassing its need for the authentication.

```
vlan 30 voice
```

#### Conclusion

Wired 802.1X offers port-level authentication and access control, ensuring only authorized devices can connect to the network. It enables dynamic VLAN assignment and eliminates the need for manual port management, enhancing network security.

In this blog post, I wanted to highlight the switch-side setup for 802.1X, focusing on the configuration process. When an unauthorized client is connected to the switch port, the system automatically assigns it to VLAN 10. Similarly, when a phone is plugged in, it seamlessly joins VLAN 30 without any additional setup. Moreover, authorized clients are automatically placed in VLAN 10.

In summary, wired 802.1X offers port-level authentication and access control, ensuring only authorized devices can connect to the network. It enables dynamic VLAN assignment and eliminates the need for manual port management, enhancing network security.

---
_Photo by Thomas Jensen @ [Unsplash.com](https://unsplash.com/fr/photos/ISG-rUel0Uw?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText)_