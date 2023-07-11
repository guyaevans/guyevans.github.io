---
title: 802.1x Port Authentication with ARUBA switches
date: 2023-07-11T12:34:34.427Z
---
## 802.1x - Why?

In the enterprise, we often use 802.1x to secure access to wired and wireless networks. For wired networks 802.1x allows us to require authentication at the port level on a supported switch which also allows us to dynamically assign VLANs to the port or connecting device.

Imagine a situation where you have a public meeting room equipped with accessible Ethernet ports, and you want to ensure secure access to those ports. In such a scenario, you have two options to consider:

1. The first option is to disable any unused ports. While this approach provides some level of security, it comes with a drawback. What if someone needs to use the port? In that case, you would have to manually log in to the switch, enable the port temporarily, and then disable it again once it's no longer in use. This process can be quite cumbersome and timewaster for IT staff.

1. The second and more efficient option is to implement 802.1X. This standard offers a superior solution by keeping the ports active at all times. With 802.1X, when a company device is plugged into the port, it will automatically gain access to the company network on the assigned VLAN. However, if an unknown device is connected, it will be assigned to a VLAN of your choice, typically a public network with only internet access. This segregation ensures that unauthorized devices cannot access the company network. The best part is that you no longer need to manually enable and disable ports whenever access is required.

## What do we need?

* A RADIUS server - we are using Microsoft NPS
* A Managed Switch - Could be any any managed switch which can do port authentication, I'm doing this on an ARUBA 2930F (which is a nice switch by the way)
* Some VLANs
  * Company VLAN: 10 - Public VLAN: 20
* A Windows computer




