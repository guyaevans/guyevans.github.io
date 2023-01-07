# BGP IPv4 Config on Mikrotik RouterOSv7

So I know a couple people who run their ASNs on Mikrotik routers (running RouterOS) as their hardware is cheap and easy to find. That includes me, I run BGP on their CHR image to announce my own ASN and IP v4 and V6 subnets. RouterOS CHR which is an image of RouterOS that you can run on your own hardware.

You would prevously use RouterOS version 6, but now version 7 has a couple stable releases and is supposed to be more stable and faster for BGP, I thought I would post some of my configs for version 7 as the config syntax has changed quite a bit. Today I'll focus on IPv4 but IPv6 is pretty much the same.

### Wait! ASN, BGP, You've lost me ...

#### What is BGP?

> Border Gateway Protocol (BGP) is the postal service of the Internet. When someone drops a letter into a mailbox, the Postal Service processes that piece of mail and chooses a fast, efficient route to deliver that letter to its recipient[^cloudflarebgp]

[^cloudflarebgp]: Source: What is BGP? | BGP routing explained - [Cloudflare](https://www.cloudflare.com/learning/security/glossary/what-is-bgp/)

#### What is an autonomous system and ASNs?
>The Internet is a network of networks*, and autonomous systems are the big networks that make up the Internet. More specifically, an autonomous system (AS) is a large network or group of networks that has a unified routing policy. Every computer or device that connects to the Internet is connected to an AS. 

> Each AS is assigned an official number, or autonomous system number (ASN), similar to how every business has a business license with an unique, official number. But unlike businesses, external parties often refer to ASes by their number alone.
[^cloudflareasn]

[^cloudflareasn]: Source : What is an autonomous system? | What are ASNs? - [Cloudflare](https://www.cloudflare.com/learning/network-layer/what-is-an-autonomous-system/)

![That's pretty slick gif](https://media4.giphy.com/media/U09CkmV0ewDbfGgAZR/giphy.gif?cid=790b7611acdfb9df5539d54ed990cfdf2d9c2469aaf1eee6&rid=giphy.gif&ct=g)

### What does the config look like?
Here are my config exports to announce one subnet via AS 211486. As you can see, before doing anything you need to add your prefixes that you want to announce to a network list which you will specify in the bgp template. The rest should be pretty self explanitory but I've detailed the main points below.

#### Network List

```sh
/ip firewall address-list
add address=185.133.194.0/24 list=bgp
```


#### Templates
ROSv7 also introduced the concept of templates to BGP, you could define one template for iBGP, one for IXPs and another for eBGP

```sh
/routing bgp template
add as=211486 disabled=no input.filter=bgp-in name=as211486-public-out-ip4 output.filter-chain=bgp-out .network=bgp \
    routing-table=main
add address-families=ipv6 as=211486 disabled=no name=as211486-public-out-ip6 output.network=bgp routing-table=main
```
#### Filters
As with any BGP setup we have filters. Mikrotik changed the filter syntax in ROSv7, it feels quite a bit like bird. Their [reference](https://help.mikrotik.com/docs/display/ROS/Route+Selection+and+Filters#RouteSelectionandFilters-FilterSyntax) is pretty good.
Here is a basic set of incoming and outgoing filters. The first two lines allow any network routes under 0.0.0.0/0 but not 185.133.194.0/24 which is mine. The third line allows my prefix the be annonced and nothing else. If no filter is defined for a route it will be rejected. 

```sh
/routing filter rule
add chain=bgp-in comment="STD IPv4 IN Filter" disabled=no rule="if (dst in 0.0.0.0/0) {set bgp-local-pref 90; accept}"
add chain=bgp-in comment="STD IPv4 IN Filter" disabled=no rule="if (dst in 185.133.194.0/24) {reject}"
add chain=bgp-out comment="STD IPv4 OUT Filter" disabled=no rule="if ( dst == 185.133.194.0/24 ) {accept}"
```

#### Connection
Lastly we add the connection which references the template, network list and filters we have just created.

```sh
/routing bgp connection
add as=211486 disabled=no input.filter=bgp-in local.role=ebgp name=ip4-mw output.filter-chain=bgp-out .network=bgp \
    remote.address=80.67.167.166/32 router-id=185.133.194.1 routing-table=main templates=as211486-public-out-ip4
```

#### Blackhole route

One thing I often forget is to add a blackhole route to my routing table for the prefixes I would like to announce. Without this, you may have the correct filters be nothing will be announced as the router will only announce prefixes in its routing table.

```sh
/ip route
add blackhole disabled=no dst-address=185.133.194.0/24
```

Congratulations! You are now announcing a network with BGP on Mikrotik RouterOSv7

![congrats](https://i.giphy.com/media/D1wxiYaBi6euR8W0VX/giphy.webp)
