---
title: Daikin AC, Home Assistant and Local control
date: 2024-09-15T21:09:43+02:00
tags:
  - Home-Assistant
  - IoT
  - AC
rssFullText: true
featuredImage: ac-workingimage.webp
lightgallery: true
---
# Air Conditioning and Home Assistant
When we bought our apartment, the previous owners had installed a Daikin minisplit airconditioner in the living room. Great, another thing to 'smartify' with Home Assistant.

I first did what I always have done in previous places and used a Broadlink Mini and the SmartIR intergration to enable smart control of the minisplit using Home Assistant. IR control is great but is lacking in some features, mainly feadback. If your device misses the IR command, well tough, they are now out of sync and you are none the wiser, and your AC did not turn on ...

## Daikin Modules and a random find

Daikin does make a wifi module to be added to the mini split but it has two major shortcomings:
* the price : I found them for 90 Euros - way to much ...
* cloud control only: The Daikin app used to have local control but after a couple of updates they have moved to full cloud control meaning that without an Internet connection you can say bye bye to your aircon ...

During my research for the Daikin module, I came across the following links: [Need Daikin Wifi? Use the Open-Source Faikin ESP32 Hardware instead of the official wifi Modules](https://community.home-assistant.io/t/need-daikin-wifi-use-the-open-source-faikin-esp32-hardware-instead-of-the-official-wifi-modules/644370) and this github [ESP32-Faikin](https://github.com/revk/ESP32-Faikin) repo by "RevK'.

The promise is great: code and a PCB to enable plugin and play wifi function with HomeAssistant via MQTT. Better yet the PCB is sold on [Amazon UK](https://www.amazon.co.uk/dp/B0C2ZYXNYQ) and based on the venerable ESP32.

I decided to buy one along with some header wires, after all ... 

{{<image src="/img/clarkson-how-hard.gif">}}

## Not very ...

I received the PCB and header wires rather quickly, thanks Amazon

{{<image src="/posts/2024-09-15_daikin-ac-home-assistant-and-local-control/faikin_pcb.jpg" class="center" width="40%">}}

Now comes the fun part, hooking it all up. But first a warning

{{< admonition type=danger title="Electricity is dangerous and can kill you">}}
Turn off your main breaker before taking apart your AC unit, if unsure call a professional.
*Don't say I didn't warn you when you get zapped or if you break something*{{< /admonition >}}