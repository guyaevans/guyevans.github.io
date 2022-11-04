---
title: VPS to a coloed server, my homelab journey
date: 2022-11-04T21:01:45.853Z
---
As a young child I always had an interest in computers and tech, little did I know that it would foundation for my carear path that I have chosen today.

So where I am retracing my steps of how I ended up with a server colocated in a datacenter that made me the sys and net admin I am today

## How I Started
Age 12 - I had built my own website (but thats a story for another post.)

At age 19, I had already learnt about running small business networks and managed a couple for family businesses and friends but I had no equipmenent or anything to learn and try things on. Thats when I learnt what a Homelab is.

### What is a Homelab?
I can hear you in the back ... What is a homelab?
> A homelab, in the simplest terms is a sandbox that you can learn and play with new or unfamiliar technologies.  They can be as simple as a set of VM’s on an old PC or laptop to as complex whole network with it's own ASN (we will get to that, eventually ...)

### Virtual Private Servers with OVH

#### VPS Number 1: The website
After doing some research I new two things:

1. What service I wanted to Hhst: my website
1. The rough location I wanted it hosted in: France

A bit more research later I found [OVH](https://ovh.fr), they are a web host based in france. OVH offered and still do cheap and very good Virtual Private Servers. I went and chose a 1vCore and 512 Mb RAM server to run my website that was based on Wordpress[^wordpress]. 

This VPS[^vps] eventually ended up hosting the website for EvansMedia, my IT and Web services company until the 10th March 2021 and what I will call *'The Great Fire of OVH*'[^ovhfire], where their four Strasbourg Datacenters went up in smoke (literally).

[^wordpress]: [Wordpress](https://wordpress.org) is a free and open-source content management system. In 2021, 30% of the worlds webites ran on Wordpress

[^vps]: Virtual Private Server
[^ovhfire]: *10 March 2021* - Millions of websites offline after fire at French cloud services firm - [Reuters](https://www.reuters.com/article/us-france-ovh-fire-idUSKBN2B20NU)
#### VPS Number 2: The storage server
