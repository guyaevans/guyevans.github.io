---
title: "Crowdsec Blocklists and Mikrotik Firewall"
date: 2025-04-24T00:00:00
rssFullText: true
lightgallery: true
featuredImage: feature.webp
authors: ["Guy Evans"]
tags:
    - security
    - crowdsec
    - mikrotik
    - selfhosted
    - Homelab
# series: 
#     - Crowdsec
summary: "One question you always get when selfhosting is: \n\n How do you secure your public facing services ? The answer usually is in layers. One layer I use is the firewall on my Mikrotik router that sits in front of all my selfhosted services. Another layer I use is Crowdsec which is an IDS/IPS that you can selfhost."
draft: true
---
One question you always get when selfhosting is:
> How do you secure your public facing services ?

The answer usually is in layers. 
One layer I use is the firewall on my Mirotik router that sits in front of all my selfhosted services. Another layer I use is Crowdsec which is an IDS/IPS that you can selfhost and is augmented by community blocklists. Today we will be setting both up so that the Mikrotik can use Crowdsec's blocklists to secure my homelab network directly at the entrance.

> CrowdSec is an open-source, collaborative security solution designed to detect and mitigate threats at scale. It functions as an Intrusion Prevention System (IPS) and Intrusion Detection System (IDS) by analyzing logs from various sources (such as firewalls, servers, and applications) to identify suspicious behavior.
>
> We won't be be covering the installation today but they do have a great [Getting Started](https://docs.crowdsec.net/docs/getting_started/) guide.
## Setup
### Install the Blocklist mirror bouncer
You can find out more about the Blocklist Mirror Bouncer at the official documentation site https://doc.crowdsec.net/docs/bouncers/blocklist-mirror. 

Install the Blacklist mirror bouncer on your crowdsec host.

```bash
sudo apt install crowdsec-blocklist-mirror

# Already installed in my case
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
crowdsec-blocklist-mirror is already the newest version (0.0.4).
0 upgraded, 0 newly installed, 0 to remove and 18 not upgraded.
```

We can then check that the bouncer has been installed and is found by Crowdsec

```bash
sudo cscli bouncers list
─────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
 Name                            IP Address   Valid  Last API pull         Type                       Version                                                                  Auth Type
─────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
 cs-firewall-bouncer             127.0.0.1    ✔️     2025-02-23T12:46:53Z  crowdsec-firewall-bouncer  v0.0.31-debian-pragmatic-amd64-4b99c161b2c1837d76c5fa89e1df83803dfbcc87  api-key
 cs-blocklist-mirror-1738504976  127.0.0.1    ✔️     2025-02-23T12:46:46Z  crowdsec-blocklist-mirror  v0.0.4-debian-pragmatic-amd64-cb1396da47b875b74486803272c91a76ef77ac7f   api-key
─────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
```
This command lists all the bouncers installed on your system. You should have a similar output.

Now we need to edit the bouncers configuration in ```/etc/crowdsec/bouncers/crowdsec-blocklist-mirror.yaml``` 

```bash
sudo vi /etc/crowdsec/bouncers/crowdsec-blocklist-mirror.yaml
```

The first setting we want to change is  ```format: mikrotik``` this sets the output format to be compatible with mikrotik cli. Finally we need to change the ```listen_uri:``` to match our local ip and the port we would like the bouncer to be available on.

Once all the changes are made your ```crowdsec-blocklist-mirror.yaml``` should look like below

```yaml {open=true}
config_version: v1.0
crowdsec_config:
  lapi_key: <API KEY>
  lapi_url: http://127.0.0.1:8080
  update_frequency: 10s
  include_scenarios_containing: []
  exclude_scenarios_containing: []
  only_include_decisions_from: []
  insecure_skip_verify: false

blocklists:
  - format: mikrotik # Supported formats are either "plain_text" or "mikrotik"
    endpoint: /security/blocklist
    authentication:
      type: none # Supported types are either "none", "ip_based" or "basic"
      user:
      password:
      trusted_ips: # IP ranges, or IPs that don't require auth to access this blocklist
        - 127.0.0.1
        - ::1

listen_uri: Server_IP:41412 # Add your server's IP
tls:
  cert_file:
  key_file:

metrics:
  enabled: true
  endpoint: /metrics

# logging configuration
log_media: file
log_dir: /var/log/
log_level: info
log_max_size: 40
log_max_age: 30
log_max_backups: 3
compress_logs: true
# enable access log of the HTTP server
enable_access_logs: true
```
We can then restart the blocklist mirror bouncer and check that the service is running

```bash
sudo systemctl restart crowdsec-blocklist-mirror
```
```bash
sudo systemctl status crowdsec-blocklist-mirror

● crowdsec-blocklist-mirror.service - CrowdSec Blocklist Mirror
     Loaded: loaded (/etc/systemd/system/crowdsec-blocklist-mirror.service; enabled; vendor preset: enabled)
     Active: active (running) since Sun 2025-02-23 13:58:54 CET; 58s ago
    Process: 2578810 ExecStartPre=/usr/bin/crowdsec-blocklist-mirror -c /etc/crowdsec/bouncers/crowdsec-blocklist-mirror.yaml -t (code=exited, status=0/SUCCESS)
   Main PID: 2578818 (crowdsec-blockl)
      Tasks: 13 (limit: 11923)
     Memory: 37.5M
        CPU: 422ms
     CGroup: /system.slice/crowdsec-blocklist-mirror.service
             └─2578818 /usr/bin/crowdsec-blocklist-mirror -c /etc/crowdsec/bouncers/crowdsec-blocklist-mirror.yaml

Feb 23 13:58:54 web01 systemd[1]: Starting CrowdSec Blocklist Mirror...
Feb 23 13:58:54 web01 systemd[1]: Started CrowdSec Blocklist Mirror.
```

Our list published by the blocklist mirror bouncer is now available at http://server_IP:41412/security/blocklist and if we visit the url in a browser or use curl we can see the list.

```bash
curl http://server_IP:41412/security/blocklist
```
```bash
/ip/firewall/address-list/remove [ find where list="CrowdSec" ];
:global CrowdSecAddIP;
:set CrowdSecAddIP do={
    :do { /ip/firewall/address-list/add list=CrowdSec address=$1 comment="$2" timeout=$3; } on-error={ }
}

/ipv6/firewall/address-list/remove [ find where list="CrowdSec" ];
:global CrowdSecAddIPv6;
:set CrowdSecAddIPv6 do={
    :do { /ipv6/firewall/address-list/add list=CrowdSec address=$1 comment="$2" timeout=$3; } on-error={ }
}
$CrowdSecAddIP "1.11.89.43" "crowdsecurity/ssh-slow-bf" "155h1m52s"
$CrowdSecAddIP "1.116.238.156" "crowdsecurity/http-probing" "161h1m51s"
$CrowdSecAddIP "1.117.113.68" "crowdsecurity/ssh-slow-bf" "155h1m52s"
$CrowdSecAddIP "1.117.37.207" "crowdsecurity/http-bad-user-agent" "154h1m52s"
$CrowdSecAddIP "1.117.73.212" "crowdsecurity/http-bad-user-agent" "136h33m13s"
$CrowdSecAddIP "1.12.48.131" "crowdsecurity/http-probing" "164h1m51s"
$CrowdSecAddIP "1.12.55.42" "crowdsecurity/http-probing" "165h1m51s"
$CrowdSecAddIP "1.14.105.106" "crowdsecurity/http-probing" "165h1m51s"
...
```
Notice how it is formatted to be pasted into the Mikrotik CLI. 
We could stop here and just past the list to the CLI and setup a block rule. But what about updates as the list is regularly updated?

### Mikrotik

*Note: we are using the Mikrotik CLI to set this up but you can also use Winbox if you wanted to.*

We now need a script to update the blocklist on our Mikrotik and a block rule to use it.

To create a script we are going to use Mikrotik's [scripting language](https://help.mikrotik.com/docs/spaces/ROS/pages/47579229/Scripting).

We can add our script using the following command while making sure to replace ```server_IP``` with the IP address of our mirror bouncer.

```sh
/system script
add dont-require-permissions=no name=crowdsec_import policy=\
    ftp,reboot,read,write,policy,test,password,sniff,sensitive,romon source=":local name \"[crowdsec]\"\
    \n:local url \"http://server_IP:41412/security/blocklist\"\
    \n:local fileName \"blocklist.rsc\"\
    \n:log info \"\$name fetch blocklist from \$url\"\
    \n/tool fetch url=\"\$url\" mode=http dst-path=\$fileName\
    \n:if ([:len [/file find name=\$fileName]] > 0) do={\
    \n    :log info \"\$name import;start\"\
    \n    /import file-name=\$fileName\
    \n    :log info \"\$name import:done\"\
    \n} else={\
    \n    :log error \"\$name failed to fetch the blocklist\"\
    \n}"
```

Next we need to add a scheduled task to run the script and update the list, we run it every 15 minutes.

```sh
/system scheduler
add interval=15m name=schedule1 on-event="system/script/run crowdsec_import"
```

Once our script has run, we can check if our address list has been populated with IPs to block.

```sh {open=true}
/ip/firewall/address-list/print where list=CrowdSec
Flags: X - disabled, D - dynamic
 #   LIST                           ADDRESS                                             CREATION-TIME        TIMEOUT
14 D ;;; http:scan
     CrowdSec                       1.12.48.131                                         2025-04-24 11:09:16  3h52m57s
15 D ;;; http:scan
     CrowdSec                       1.12.55.42                                          2025-04-24 11:09:16  4h52m57s
16 D ;;; http:scan
     CrowdSec                       1.14.105.106                                        2025-04-24 11:09:16  5h52m57s
17 D ;;; http:scan
     CrowdSec                       1.14.14.169                                         2025-04-24 11:09:16  5h52m57s
18 D ;;; http:scan
     CrowdSec                       1.14.61.226                                         2025-04-24 11:09:16  3h52m57s
19 D ;;; http:scan
     CrowdSec                       1.14.64.103                                         2025-04-24 11:09:16  2d16h52m57s
20 D ;;; http:scan
     CrowdSec                       1.14.66.207                                         2025-04-24 11:09:16  52m57s
21 D ;;; http:scan
     CrowdSec                       1.14.73.93                                          2025-04-24 11:09:16  5h52m57s
22 D ;;; http:scan
     CrowdSec                       1.14.77.81                                          2025-04-24 11:09:16  3h52m57s
23 D ;;; http:scan
     CrowdSec                       1.14.92.22                                          2025-04-24 11:09:16  3h52m57s
24 D ;;; http:scan
     CrowdSec                       1.14.93.149                                         2025-04-24 11:09:16  5h52m57s
25 D ;;; http:scan
     CrowdSec                       1.14.94.175                                         2025-04-24 11:09:16  5h52m57s
26 D ;;; http:scan
     CrowdSec                       1.14.96.114                                         2025-04-24 11:09:16  4h52m57s
27 D ;;; http:scan
-- [Q quit|D dump|down]
```

We finally need to add a firewall rule to actually block any IPs in our list. We add two rules, one for the forward chain and another for the input chain. We add these rules at the top of our list as the firewall rules a processed in top down order.

```sh
/ip firewall filter
add action=drop chain=forward comment="Drop from Crowdsec Lists" src-address-list=CrowdSec
add action=drop chain=input comment="Drop from Crowdsec Lists" src-address-list=CrowdSec
```

Once our rule has been added we can see traffic being blocked.

```sh
# CHAIN    ACTION      BYTES  PACKETS
;;; Drop from Crowdsec Lists
0 forward  drop    1 524 773   25 568
;;; Drop from Crowdsec Lists
1 input    drop      339 980    6 586
```

Now the firewall on our Mikrotik router is using address lists provided by our Crowdsec bouncer to block suspicious requests.

## End

This just a small taste of what we can do with CrowdSec. 
CrowdSec can do a lot more, like acting as a Web Application Filter and virtual patching in front of your websites, scanning system logs for suspicious behaviors, and then block those behaviors. I might write a poste about its other features in the future.