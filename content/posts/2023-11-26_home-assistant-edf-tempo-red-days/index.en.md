---
title: Home Assistant - EDF Tempo - Automations for Red days
date: 2023-11-26T18:07:38.226Z
---

## EDF Tempo - A Electricity Tariff
In France, our electricity supplier EDF offers a couple of contract types. "Base" which is your normal fixed rate contract; "Heures Creuses" which has a different tariffs for peak and off peak usage. And finally the contract that I have chosen which is "Tempo", this contract is a bit different as you have three different possible peaks tariffs and off peak tariffs.

The tarriffs are laid out as follows over the year:
* 22 Red days (in the winter period between November 1st and March 30th) - where the price is notably higher during peak hours (almost 0.70 centimes per kwh)
* 43 White days - where the price is average during peak hours
* remaining days are Blue - the price is the cheapest on these days.

The days is split into a peak tariff (6h to 22h) and and off peak tariff 22h to 6h). The Red and White days can only fall on week days so the rest of the time you are on the cheapest tariff. The day's colour is announced the day.

So to recap:

__*If*__ you can limit your electricity usage 22 days per year, this contract will net you some good savings over the course of the year. And this is what we are going to use some automations for ...

Bonus: the off peak tarrif for a red day is still cheaper that the normal off peak tarriff