---
title: VPS to a coloed server, my homelab journey - Part 2
date: 2022-12-24T15:02:35.756Z
featuredImage: cbo-rack-mw2.jpg
series: 
  - VPS to a coloed server my homelab journey
---

## After the Virtual Private Servers

After loosing one of my virtual private servers along with the data in the _'The Great Fire of OVH_ :fire:'[^ovhfire] (Yes I know I should have had backups of that VPS[^vps]), I decided why not setup a Proxmox[^pve] Cluster[^cluster] on a couple dedicated servers. 

So I went ahead a found [OneProvider.com](https://oneprovider.com) where you can rent dedicated servers for cheap, their cheap hardware is old and the connectivity is ok but you cannot beat there pricing (Server with an Intel Atom C2750, 16 Gb Ram, 1Tb Hard drive and 1Gb of 'unlimited' bandwith was €14 per server/month at the time). That went great for a couple of months but I kept getting fustrated with not having full control over the hardware and my growing interest for BGP[^bgp] and how the internet is really just a bunch of inter connected networks. After some trial and error I decided I should review my home-lab goals and re-think my shopping list.

## The Shopping List

So what did I want?
- My own (recent-ish) hardware
- A Decent Host for my hardware (Colocation Provider)
- BGP transit connection with my own leased /24 block of IPs
  
Bonus:
- 10 Gb connection to the internet

I now had a plan and a decent shopping list, although we didn't talk about budget. Lets just say around €700 for the hardware and another €100-120 per month for the hosting.

### A Decent Host for my hardware

I first found a great host for my hardware while browsing [LaFibre.info](https://lafibre.info/) a french forum about fiber to the home and IT in general. The host I chose is [*MilkyWan*](https://milkywan.fr), they are a small not for profit internet service provider run by a small passionate team. 

They specialise in providing Virtual Servers, Hosting, ADSL and Fiber Internet. They also have multiple points of presence in french datacenters where they can host hardware and provide BGP transit. They also have a great community on telegram.

Cost : €75 for 1U Server Housing, Power and Network Connection.

Bonus: They can provide a 10Gb connection to the internet.

_Did I mention they have a great community, being charity run means that they hold annual meeting and have the best interests of their members at heart. Since the end of 2022 I am also an elected representative of the members to the charity comity_

### My own (recent-ish) hardware

Now I had found somewhere for my hardware, I actually needed some hardware. I found what I needed in [BargainHardware.co.uk](https://www.bargainhardware.co.uk/), they specialise in secondhand enterprise hardware and have great prices.

Here is the server I settled on:

**Dell Poweredge R620**
- 2 x Intel Xeon E5-2660 V2 2.20Ghz Ten Core CPUs
- 96 Gb of DDR3 RAM
- 2 x 240 Gb Crucial SSDs for the boot drives (They will be in Raid 1 to provide redundancy in case of a drive failure)
- 6 x 900 Gb SAS Hard drives for the datadrives to give us a total of 5Tb usable space (This will be the main storage pool and will be in a ZFS pool to provide redundancy in case of a drive failure)
- 1 x Dell BCM57800S Quad Port - 1GbE, 10GbE SFP+, RJ45 network card

Cost: €700 including shipping to france.

Not too old to be inefficent to run and not too recent that it breaks the bank. This should provide me with plenty of horsepower to multiple virtual machines.


{{< series_en "VPS to a coloed server my homelab journey" >}}

[^ovhfire]: *10 March 2021* - Millions of websites offline after fire at French cloud services firm - [Reuters](https://www.reuters.com/article/us-france-ovh-fire-idUSKBN2B20NU)
[^vps]: Virtual Private Server
[^pve]: Proxmox Virtual Environment (PVE) is a free and open source virtualization solution that allows the deployment of virtualized systems
[^cluster]: A computer cluster is a set of computers that work together so that they can be viewed as a single system
[^bgp]: Border Gateway Protocol (BGP) is the routing protocol for the Internet. Much like the post office processing mail, BGP picks the most efficient routes for delivering Internet traffic. [What is BGP - Cloudlare](https://www.cloudflare.com/learning/security/glossary/what-is-bgp/)