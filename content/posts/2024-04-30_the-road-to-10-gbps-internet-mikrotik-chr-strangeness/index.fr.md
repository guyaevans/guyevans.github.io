---
title: "Vers l'internet 10 Gbps - Bizarrerie Mikrotik CHR "
date: 2024-04-30T20:12:10.480Z
featuredImage: feature.jpeg
---
## Le début
Mon serveur [^Homelab] qui se trouve dans baie réseau de [Milywan](https://milkywan.fr) à Croissy-Beaubourg/CBO et dispose d'une carte réseau 10G, car c'était l'une de mes exigences lors de sa mise en place. Une chose que je n'ai jamais vraiment utilisée à son plein potentiel. 

__Aujourd'hui, cela a changé__ _(en quelque sorte)._

[^Homelab]: [J'ai posté deux articles](https://guy-evans.com/series/vps-to-a-coloed-server-my-homelab-journey/) sur le passage d'un serveur virtuel et à mon propre serveur dans un rack colo.

Milkywan a un groupe de discussion sur Telegram où nous parlons de technologie et d'autres choses entre membres. L'un des membres du groupe cherchait quelqu'un ayant un routeur Mikrotik "on-net" pour effectuer des tests de bande passante. 

- J'essaie de comprendre pourquoi et si je suis limité à 1G.

Mikrotik dispose d'un outil intégré pour cela :
> Le Bandwidth Tester peut être utilisé pour mesurer le débit vers un autre routeur MikroTik (avec ou sans fil) et ainsi aider à découvrir les "goulots d'étranglement" du réseau. - [Mikrotik](https://help.mikrotik.com/docs/display/ROS/Bandwidth+Test)


J'ai proposé mon instance CHR </br></br></br>
{{<figure src="/img/jack_whatcouldgowrong.gif">}}
## Un bon début
Je lui ai donc envoyé par dm les détails de la connexion et il a commencé ses tests !

[^1]: Cloud Hosted Router (CHR) est une version de RouterOS destinée à fonctionner en tant que machine virtuelle. Elle prend en charge l'architecture x86 64 bits et peut être utilisée sur la plupart des hyperviseurs populaires tels que VMWare, Hyper-V, Proxmox, etc. - [Mikrotik - Help](https://help.mikrotik.com/docs/display/ROS/Cloud+Hosted+Router%2C+CHR)

{{< figure src="IMG_4388.jpeg" caption="Transmission à 2,7G - pas mal !">}}

Pas mal pour un vieux R620 - Après quelques changements - mon collègue a fait un autre test ...

Nous avons réussi à avoir 4G. (J'ai oublié de faire une capture d'écran) Ok mais toujours pas tout à fait notre objectif.

## Le goulot d'étranglement

J'ai alors remarqué quelque chose ...

{{<figure src="IMG_4389.jpeg" caption="AH ! - Un problème">}}

J'avais donné que 4 vCPUs à mon instance CHR, pourquoi aurais-je besoin de plus ? Mais avec nos tests, nous les utilisions au maximum. Solution simple à ce problème : ajouter plus de vCPUs. J'ai donné à la VM 10 vCPU et je l'ai redémarrée. 

Un autre test plus tard, selon la weathermap[^wm] de Milkywan cela nous a permis d'atteindre 5G, mais nous voulions plus. 

{{<figure src="IMG_4390.jpeg" caption="YVous pouvez voir 5Gbps provenant du routeur nommé cer2024.edge.tls (en bas au milieu).">}} 

[^wm]: Un Weathermap est une carte qui affiche le réseau des opérateurs avec des statistiques superposées.

## Un ban et une bizarrerie

Tout en discutant de quelques idées supplémentaires, nous avons laissé le test se dérouler jusqu'à ce que nous recevions un message dans le chat du groupe :

- _Hé les gars, vous déclenchez des alertes sur nos systèmes de surveillance_ 🤣 
_-L'un des admins_

Oui, un trafic constant de 5 Gbps incitera le NOC d'un petit fournisseur d'accès à regarder ce qui se passe. 

J'ai donc décidé d'ajouter un autre socket de 12 vCPUs pour un total de 24 vCPUs en pensant que **plus de cpu = plus de bande passante.**
{{<figure src="/img/jeremy-clarkson-sometimes-my-genius.gif">}}
C'est là que j'ai trouvé une bizarrerie avec CHR, il ne reconnaît pas un deuxième socket CPU. Donc, un autre redémarrage plus tard, nous avions un cpu à socket unique avec 24 vCPUs. Ce qui nous a permis d'atteindre 6G. 

Pendant que mon collègue effectuait d'autres tests en parallèle, nous avons reçu un autre message :

— *Les gars? Vous venez de déclencher l'anti-DDoS*. 😅


Cela nous a bloqués pendant 15 minutes. Mais après cela, nous avons effectué un autre test et nous avons obtenu ... 6 G ... à nouveau. 

Nous avons également découvert après coup que nous avions déclenché la mitigation DDOS sur un autre des routeurs de Milkywan. 

Nous avons arrêté après parce que nous pouvions continuer à optimiser toute la nuit, déclencher plus de mitigation DDOS et n'arriver à rien de plus.

Je suis quand même content, mon petit R620 dans la colo peut tirer un trafic décent de l'internet. J'attendrai la prochaine occasion pour aller chercher les 4Go manquants. 