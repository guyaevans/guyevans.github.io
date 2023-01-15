---
title: Self-hosted Email - Why and How?
date: 2023-01-09T20:22:58.009Z
featuredImage: brett-jordan-lpzy4da9aro-unsplash.jpg
summary: "Question: What is one service on the internet that we use regularly and has become part of our day to day? **EMAIL** - 
Chances are that is you use a computer, you have at the very leat ine email address, some people have more that one (I've lost count of how many I have). Why should you have your own?"
tags:
  - email
  - self-hosted
  - mailcow
  - guide
---
## Email
Question: What is one service on the internet that we use regularly and has become part of our day to day? **EMAIL**

Chances are that if you use a computer, you have at the very least one email address, some people have more that one (I've lost count). That brings me to a firm belief of mine: if you work in IT in _any_ shape or form, your email should be your own, with your own domain name _(hello@evans.fr for example)_ instead of using a @gmail.com @outlook.com or @aol.com _(that's an old one)_ or any other provider.

## Why (self-host)
I for one have used to host all of my email addresses on Google Apps _(now called Google Workspace)_. It was simple point the right mx records to their servers and boom your own email domain.

Many other providers also let you do this but where google was clever, is that their service had a great free tier (as in free beer) until they decided to pull the rug out from under us all and sunsetted the free tier. (Another reason people may self-host their email is having full control over the service and not sharing any data with the FAANG companies)

That got me thinking, why not host my own email someday. So to as with a lot of my research starts I go to Reddit.com and the general consensus is: Why bother. But still I continue my research and decide to continue. Afterall the best way to learn is to do it. How hard can it be?

_but first, we need coffee..._
{{<figure src="/img/bbt-first-coffee.gif">}}

## How (Using Docker & Mailcow)

I won't be deep diving into how to set an email server as there are plenty of guides to be found but I will at least point you in the right direction.

### What we need

1. A Domain name of your own
2. A VM or VPS server with: 2 VCPUs, 4 GB of RAM (at minimum), 80 GB storage (or more), a clean IP address _(I suggest [Milkywan](https://milkywan.fr/) or [Virtua.Cloud](https://www.virtua.cloud/) both are based in france and have great connectivity.)_
3. Docker & Docker Compose (I suggest using docker as tt is fast to setup)
4. [Mailcow Dockerized](https://mailcow.email/), my mail server of choice - it contains all the services needed and runs in docker containers

### Setup
#### DNS
You will need to setup the following DNS records for Mailcow to work
```
# Name              Type       Value
mail                IN A       1.2.3.4
autodiscover        IN CNAME   mail.example.org. (your ${MAILCOW_HOSTNAME})
autoconfig          IN CNAME   mail.example.org. (your ${MAILCOW_HOSTNAME})
@                   IN MX 10   mail.example.org. (your ${MAILCOW_HOSTNAME})
```
#### Installing Mailcow

Once the DNS records are propagated, you can install and setup mailcow.
First we clone the repository and set the permissions on the folders
```bash
su
umask
0022 # <- Verify it is 0022
cd /opt
git clone https://github.com/mailcow/mailcow-dockerized
cd mailcow-dockerized
```
Once that is done we need to initialise mailcow and create a configuration file using the command below, It will ask for your FQDN (domain name)
```bash
./generate_config.sh
```
Once you have answered the questions it will generate the configuration file for you. Once generated we can run mailcow using the commands below
```bash
docker compose pull
docker compose up -d
```
CONGRATULATIONS ! You have setup a mail server (well almost)

Now that the server is running, you can access https://${MAILCOW_HOSTNAME} with the default credentials ```admin``` + password ```moohoo``` and start setting up domains and email addresses.

Mailcow has a great [help site](https://docs.mailcow.email/post_installation/firststeps-ssl/) which will walk you through most of the settings that I have not covered.

## End
I hope that with this short post, you can see that it's not actually hard to setup a mailserver. What is hard is keeping of spam lists and fighting incoming spam, but for personal use and with the help of Mailcow you should be fine.

Another thing you should not forget is ... BACKUPS. 

If you host your main email on this server, you should back it up regularly.

---
_Photo by Brett Jordon @ [Unsplash.com](https://unsplash.com/@brett_jordan?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText)_