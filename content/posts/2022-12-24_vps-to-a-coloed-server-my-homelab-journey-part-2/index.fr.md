---
title: " D'un VPS à un serveur en colocation, l'evolution de mon homelab - Partie 2"
date: 2023-01-07T16:04:28.538Z
featuredImage: cbo-rack-mw2.jpg
tags:
  - homelab
  - ovh
  - vps
  - serveurs
  - wordpress
  - colo
  - colocation
---
## Après les serveurs virtuels

Après avoir perdu l'un de mes serveurs privés virtuels ainsi que les données dans le _'The Great Fire of OVH_ :fire:'[^ovhfire] (Oui je sais que j'aurais dû avoir des sauvegardes de ce VPS[^vps]), je me suis dit pourquoi ne pas mettre en place un Cluster[^cluster] Proxmox[^pve] sur quelques serveurs dédiés. 

Après quelques recherches et j'ai trouvé [OneProvider.com] (https://oneprovider.com) où vous pouvez louer des serveurs dédiés pour pas cher, leur matériel bon marché est vieux et la connectivité est correcte mais vous ne pouvez pas battre leurs prix (un serveur avec un Intel Atom C2750, 16 Go de RAM, 1 To de disque dur et 1 Go de bande passante "illimitée" coûtait 14 € par serveur/mois à l'époque). Cela s'est bien passé pendant quelques mois, mais j'étais de plus en plus frustré de ne pas avoir le contrôle total du matériel et de mon intérêt croissant pour BGP [^bgp] et la façon dont Internet n'est en fait qu'un ensemble de (petits) réseaux interconnectés. Après quelques essais et erreurs, j'ai décidé que je devais revoir les objectifs de mon home-lab et repenser ma liste d'achats.

{{< figure src="/img/going-to-need-more-coffee-bbt.gif">}}

## La liste des achats

Alors, qu'est-ce que je voulais ?
- Mon propre matériel (récent)
- Un hébergeur convenable pour mon matériel (fournisseur de colocation)
- Une connexion de transit BGP avec mon propre bloc d'adresses IP /24 louées.
  
Bonus :
- Une connexion de 10 Gb à l'internet

J'avais maintenant un plan et une liste d'achats décente, bien que nous n'ayons pas parlé du budget. Disons environ 700 € pour le matériel et 100 à 120 € par mois pour l'hébergement.

### Un Hébergeur

J'ai trouvé un excellent hébergeur pour mon matériel en parcourant [LaFibre.info](https://lafibre.info/) un forum français sur la FTTH en france et l'informatique en général. L'hébergeur que j'ai choisi est [*MilkyWan*](https://milkywan.fr), c'est un petit fournisseur de services Internet à but non lucratif géré par une petite équipe passionnée. 

Ils sont spécialisés dans la fourniture de serveurs virtuels, l'hébergement, l'ADSL et l'Internet par fibre. Ils ont également plusieurs points de présence dans des centres de données français où ils peuvent héberger du matériel et fournir du transit BGP. Ils ont également une grande communauté sur telegram.

Coût : 75 € pour un serveur 1U, une alimentation et une connexion réseau.

Bonus : Ils peuvent fournir une connexion 10Gb à internet.

_J'ai mentionné qu'ils ont une grande communauté, étant géré par une association caritative, cela signifie qu'ils tiennent une réunion annuelle et qu'ils ont à cœur les meilleurs intérêts de leurs membres et le reseau. Depuis la fin de 2022, je suis également élu comme représentant des membres de l'association._

### Mon propre matériel

Maintenant que j'avais trouvé un endroit pour mon équipement, j'avais besoin de cet équipement. J'ai trouvé ce dont j'avais besoin sur [BargainHardware.co.uk](https://www.bargainhardware.co.uk/), ils sont spécialisés dans le matériel d'entreprise d'occasion et ont des prix très intéressants.

Voici le serveur que j'ai choisi :

**Dell Poweredge R620**
- 2 x Intel Xeon E5-2660 V2 2.20Ghz CPUs Dix cœurs
- 96 Go de RAM DDR3
- 2 x 240 Go Crucial SSD pour les disques de démarrage (Ils seront en Raid 1 pour assurer la redondance en cas de défaillance d'un disque)
- 6 disques durs SAS de 900 Go pour les disques de données, ce qui nous donne un total de 5 To d'espace utilisable (il s'agira du pool de stockage principal et il sera dans un pool ZFS pour assurer la redondance en cas de défaillance d'un disque).
- 1 x carte réseau Dell BCM57800S Quad Port - 1GbE, 10GbE SFP+, RJ45

Coût : 700 €, y compris les frais d'expédition vers la France.

Pas trop vieux pour être peu rentable et pas trop récent pour faire exploser la tirelire. Cela devrait me fournir beaucoup de puissance pour de multiples machines virtuelles.

## Tout mettre en place

Après avoir commandé le serveur et l'hébergement, il ne me restait plus qu'à configurer [Proxmox] (https://www.proxmox.com) et à envoyer mon tout nouveau serveur à *Milkywan* qui me l'a installé dans son centre de données. 

Je dispose maintenant de mon propre matériel avec une connexion 10G dans un centre de données français. Vous pouvez remarquer dans l'image du speedtest ci-dessous que mon nom apparaît comme le FAI, c'est parce que je suis effectivement devenu mon propre FAI puisque je fournis à mon serveur l'internet via mon propre ASN ([AS211486](https://bgp.he.net/AS211486)) et mes IPs. (Cloudflare a un excellent article sur BGP [^bgp])

Mon serveur héberge la plupart de mon 'homelab', y compris certaines parties de ce site web, comme le système de commentaires et les statistiques. J'ai également une petite installation à la maison avec un NAS et un microserveur dont je vais probablement écrire un autre billet à ce sujet.

![10G Proof](https://www.speedtest.net/result/c/bca2e66f-818f-413c-9ac0-01f538aaf561.png " Presque 10G mais c'est une autre histoire")

Mon serveur est la dernière machine dans le rack sur l'image d'en-tête de ce post. Avez-vous voulu ou avez-vous mis en place votre propre homelab ? Partagez vos commentaires ci-dessous.

[^vps]: Serveur privé virtuel
[^ovhfire]: *10 mars 2021* - Le site de l’entreprise OVH à Strasbourg touché par un « important incendie » - [Le Monde](https://www.lemonde.fr/societe/article/2021/03/10/a-strasbourg-un-important-incendie-sur-le-site-de-l-entreprise-ovh-classe-seveso_6072548_3224.html)
[^pve]: [Proxmox Virtual Environment (PVE)](https://www.proxmox.com) est une solution de virtualisation gratuite et open source qui permet le déploiement de systèmes virtualisés
[^bgp]: Border Gateway Protocol (BGP) est le service postal d'Internet. Lorsque quelqu'un dépose une lettre dans une boîte aux lettres, le service postal traite ce courrier et choisit un itinéraire rapide et efficace pour livrer cette lettre à son destinataire