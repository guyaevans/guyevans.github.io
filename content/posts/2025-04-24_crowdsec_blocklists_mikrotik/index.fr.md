---
title: "Blocklists Crowdsec et Pare-Feu Mikrotik"
date: 2025-04-24T00:00:00
rssFullText: true
lightgallery: true
featuredImage: feature.webp
authors: ["Guy Evans"]
tags:
    - securité
    - crowdsec
    - mikrotik
    - selfhosted
    - Homelab
# series: 
#     - Crowdsec
summary: "Une question récurrente lors de l'auto-hébergement de servicee est : \n comment sécuriser vos services qui sont accessible publiquement ? La réponse se trouve généralement en couches. J'utilise notamment le pare-feu de mon routeur Mikrotik, situé devant tous mes services auto-hébergés. J'utilise également Crowdsec, un système de détection et de prévention d'intrusion (IDS/IPS) auto-hébergé."
---
ne question récurrente lors de l'auto-hébergement de servicee est :
> Comment sécuriser vos services qui sont accessible publiquement ?

La réponse est généralement en couches. J'utilise notamment le pare-feu de mon routeur Mirotik, placé devant tous mes services auto-hébergés. J'utilise également Crowdsec, un système de détection et de prévention d'intrusion (IDS/IPS) auto-hébergé, complété par des listes de blocage communautaires. Aujourd'hui, nous allons configurer les deux pour que le Mikrotik puisse utiliser les listes de blocage de Crowdsec pour sécuriser mon réseau homelab directement à l'entrée.

> CrowdSec est une solution de sécurité collaborative open source conçue pour détecter et atténuer les menaces à grande échelle. Elle fonctionne comme un système de prévention des intrusions (IPS) et un système de détection des intrusions (IDS) en analysant les journaux provenant de diverses sources (pare-feu, serveurs et applications) afin d'identifier les comportements suspects.
>
> Nous ne couvrirons pas l'installation aujourd'hui, mais ils ont un excellent [ Guide de Mise en route](https://docs.crowdsec.net/docs/getting_started/).
## Installation
### Installer le miroir de la liste de blocage
Vous pouvez en savoir plus sur le Blocklist Mirror Bouncer sur le site de documentation officiel https://doc.crowdsec.net/docs/bouncers/blocklist-mirror. 

Installer le bouncer sur votre hôte crowdsec.

```bash
sudo apt install crowdsec-blocklist-mirror

# Already installed in my case
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
crowdsec-blocklist-mirror is already the newest version (0.0.4).
0 upgraded, 0 newly installed, 0 to remove and 18 not upgraded.
```

Nous pouvons ensuite vérifier que le bouncer a été installé et qu'il est trouvé par Crowdsec

```bash
sudo cscli bouncers list
─────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
 Name                            IP Address   Valid  Last API pull         Type                       Version                                                                  Auth Type
─────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
 cs-firewall-bouncer             127.0.0.1    ✔️     2025-02-23T12:46:53Z  crowdsec-firewall-bouncer  v0.0.31-debian-pragmatic-amd64-4b99c161b2c1837d76c5fa89e1df83803dfbcc87  api-key
 cs-blocklist-mirror-1738504976  127.0.0.1    ✔️     2025-02-23T12:46:46Z  crowdsec-blocklist-mirror  v0.0.4-debian-pragmatic-amd64-cb1396da47b875b74486803272c91a76ef77ac7f   api-key
─────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
```
Cette commande répertorie tous les bouncers installés sur votre système. Vous devriez obtenir un résultat similaire.

Nous devons maintenant modifier la configuration du bouncer dans ```/etc/crowdsec/bouncers/crowdsec-blocklist-mirror.yaml``` 

```bash
sudo vi /etc/crowdsec/bouncers/crowdsec-blocklist-mirror.yaml
```

Le premier paramètre que nous voulons modifier est ```format: mikrotik``` cela définit le format de sortie pour être compatible avec mikrotik cli. Finalement, nous devons changer le ```listen_uri:``` pour correspondre à notre adresse IP locale et au port sur lequel nous souhaitons que le bouncer soit disponible.

Une fois toutes les modifications effectuées, votre ```crowdsec-blocklist-mirror.yaml``` devrait ressembler à ci-dessous

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
Nous pouvons ensuite redémarrer le bouncer miroir de liste de blocage et vérifier que le service est en cours d'exécution

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

Notre liste publiée par le bouncer est désormais disponible sur http://server_IP:41412/security/blocklist et si nous visitons l'URL dans un navigateur ou utilisons curl, nous pouvons voir la liste.

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
Notez comment elle est formatée pour être collée dans l'interface de ligne de commande Mikrotik.
Nous pourrions nous arrêter ici et simplement coller la liste dans la CLI et configurer une règle de blocage. Mais qu'en est-il des mises à jour, puisque la liste est régulièrement mise à jour ?

### Mikrotik

*Note: nous utilisons la CLI Mikrotik pour configurer ceci mais vous pouvez également utiliser Winbox si vous le souhaitez.*

Nous avons maintenant besoin d'un script pour mettre à jour la liste de blocage sur notre Mikrotik et d'une règle de blocage pour l'utiliser.

Pour créer un script, nous allons utiliser le [langage de script de Mikrotik](https://help.mikrotik.com/docs/spaces/ROS/pages/47579229/Scripting).

Nous pouvons ajouter notre script en utilisant la commande suivante en veillant à remplacer ```server_IP``` par l'adresse IP de notre bouncer miroir.

```sh
/system script
add dont-require-permissions=no name=crowdsec_import policy=\
    read,write source=":local name \"[crowdsec]\"\
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

Ensuite, nous devons ajouter une tâche planifiée pour exécuter le script et mettre à jour la liste, nous l'exécutons toutes les 15 minutes.

```sh
/system scheduler
add interval=15m name=schedule1 on-event="system/script/run crowdsec_import"
```

Une fois notre script exécuté, nous pouvons vérifier si notre liste d'adresses a été remplie avec des IP à bloquer.

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

Nous devons enfin ajouter une règle de pare-feu pour bloquer toutes les adresses IP de notre liste. Nous ajoutons deux règles : une pour la chaîne de transfert et une autre pour la chaîne d'entrée. Ces règles sont placées en haut de la liste, car elles sont traitées de haut en bas.

```sh
/ip firewall filter
add action=drop chain=forward comment="Drop from Crowdsec Lists" src-address-list=CrowdSec
add action=drop chain=input comment="Drop from Crowdsec Lists" src-address-list=CrowdSec
```

Une fois notre règle ajoutée, nous pouvons voir que le trafic est bloqué.

```sh
# CHAIN    ACTION      BYTES  PACKETS
;;; Drop from Crowdsec Lists
0 forward  drop    1 524 773   25 568
;;; Drop from Crowdsec Lists
1 input    drop      339 980    6 586
```

Le pare-feu de notre routeur Mikrotik utilise désormais les listes d'adresses fournies par notre bouncer CrowdSec pour bloquer les requêtes suspectes.

## Fin

Ceci n'est qu'un aperçu des possibilités offertes par CrowdSec.
CrowdSec peut faire bien plus, comme agir comme un filtre d'application Web et appliquer des correctifs virtuels devant vos sites Web, analyser les journaux système à la recherche de comportements suspects, puis bloquer ces comportements. Je pourrais écrire un article sur ses autres fonctionnalités dans le futur.