---
title: Daikin AC, Home Assistant and Local control
date: 2024-10-09T10:09:43+02:00
tags:
  - Home-Assistant
  - IoT
  - AC
rssFullText: true
featuredImage: feature.jpg
lightgallery: true
summary: When we bought our apartment, the previous owners had installed a Daikin minisplit AC in the living room. Great, another thing to 'smartify' with Home Assistant. But Daikin's Wifi modules are way to expensive and cloud dependent.
---
## Air Conditioning and Home Assistant
When we bought our apartment, the previous owners had installed a Daikin minisplit AC in the living room. Great, another thing to 'smartify' with Home Assistant[^HA].

[^HA]: [Home Assistant](https://www.home-assistant.io/) - Open source home automation that puts local control and privacy first. Powered by a worldwide community of tinkerers and DIY enthusiasts. 

I first did what I always have done in previous places and used a [Broadlink Mini](https://amzn.to/4eAfGEb) and the [SmartIR](https://github.com/smartHomeHub/SmartIR/) integration to enable smart control of the minisplit using Home Assistant. IR control is great but is lacking in some features, mainly feadback. If your device misses the IR command, well tough, they are now out of sync and you are none the wiser, and your AC did not turn on ...

## Daikin Modules and a random find

Daikin does make a [wifi module](https://amzn.to/47VGXyB) to be added to the mini split but it has two major shortcomings:
* the price : I found them for 50-90 Euros *(depending on the version you need)* - way to much ...
* cloud control only: The Daikin app used to have local control but after a couple of updates they have moved to full cloud control meaning that without an Internet connection you can say bye bye to your aircon ...

During my research for the Daikin module, I came across this post on the Home Assisant forum: [Need Daikin Wifi? Use the Open-Source Faikin ESP32 Hardware instead of the official wifi Modules](https://community.home-assistant.io/t/need-daikin-wifi-use-the-open-source-faikin-esp32-hardware-instead-of-the-official-wifi-modules/644370) and this github repo [ESP32-Faikin](https://github.com/revk/ESP32-Faikin) by "RevK'.

The promise is great: code and a PCB to enable plug and play wifi function with HomeAssistant via MQTT. Better yet the PCB is sold on [Amazon UK](https://www.amazon.co.uk/dp/B0C2ZYXNYQ) and based on the venerable ESP32.

I decided to buy one along with some header wires, after all ... 

{{<figure src="/img/clarkson-how-hard.gif">}}

## Hardware setup

I received the PCB and header wires rather quickly, thanks Amazon

{{<image src="/posts/2024-10-09_daikin-ac-home-assistant-and-local-control/faikin_pcb.jpg" class="center" width="40%" caption="Faikin ESP32">}}

Now comes the fun part, hooking it all up. *But first a warning* :zap: 🚨:

{{< admonition type=danger title="Electricity is dangerous and can kill you">}}
Turn off your main breaker before taking apart your AC unit, if unsure call a professional.
*Don't say I didn't warn you when you get zapped or if you break something*{{< /admonition >}}

Mini split units usually come apart rather easy. A screw or two and then a few clips hold the chassis on.

{{<image src="/posts/2024-10-09_daikin-ac-home-assistant-and-local-control/naked_split.jpg" class="center" width="90%" caption="And look a naked minisplit">}}

Once the AC has it's cover off we need to look for the main circuit board. Usually located on the side. And for my AC least the plug we need to use is labelled S21. **You don't want to touch anything else in here. Especially any capacitors :zap:**

{{<image src="/posts/2024-10-09_daikin-ac-home-assistant-and-local-control/pcb_circuit.jpg" class="center" width="90%" caption="The AC circuit board and the installed Faikin PCB - **Don't touch that capacitor**">}}

Once installed, we can put the metal covering back on. And there is a perfect space for the Faikin PCB below.

{{<image src="/posts/2024-10-09_daikin-ac-home-assistant-and-local-control/installed_pcb.jpg" class="center" width="90%" caption="Final installation of the PCB">}}

After putting all the covers back on the AC unit we can turn our main breaker back on.

*You did turn it off right?* :eyes:


Once your AC has it's power back you will be able to connect to the Faikin via wifi

{{<image src="https://github.com/revk/ESP32-Faikin/blob/main/Manuals/WiFi1.png?raw=true" width="90%" caption="Choose the wifi broadcast by the PCB">}}

It will then ask you to enter your wifi network and MQTT server details

{{<image src="https://github.com/revk/ESP32-Faikin/blob/main/Manuals/WiFi3.png?raw=true" width="90%" caption="Enter your wifi and MQTT details">}}

Once done you will be presented with some AC controls and guess what they work instantly.

{{<image src="https://github.com/revk/ESP32-Faikin/blob/main/Manuals/Controls.png?raw=true" width="90%" caption="AC Commands">}}

Finally we need to setup the Home Assistant side.

## Home Assistant

Not much to do here. If your MQTT broker is setup correctly in Home Assistant. Your AC unit should show up on it's own.

{{<image src="/posts/2024-10-09_daikin-ac-home-assistant-and-local-control/ha1.png" class="center" width="90%" caption="AC unit in Home Assistant">}}

{{<image src="/posts/2024-10-09_daikin-ac-home-assistant-and-local-control/ha2.png" class="center" width="90%" caption="AC controls in Home Assistant">}}

With that we now have full **local** control over our AC unit which includes feedback of the AC's state to Home Assistant.

No more wondering why the room is warm and why the AC did not turn on.

---
_Note: some links may be affiliate links. As an Amazon Associate I may earn a small commission from qualifying purchases at no extra cost to the buyer._