---
title: " D'un VPS à un serveur en colocation, l'evolution de mon homelab "
date: 2022-11-04T21:01:45.853Z
---
J'ai toujours eu un intérêt pour les ordinateurs et la technologie. J'étais loin de me douter qu'il s'agirait de la base de mon parcours professionnel.

C'est pourquoi je souhaite retracer mes pas et expliquer comment je me suis retrouvé avec un serveur situé dans un centre de données qui a fait de moi l'administrateur système et réseau que je suis aujourd'hui.

## Comment j'ai commencé
A 12 ans, j'avais construit mon propre site web (mais c'est une histoire pour un autre article).

À 19 ans, j'avais déjà appris à gérer des réseaux de petites entreprises et j'en ai géré quelques-uns pour des entreprises familiales et des amis, mais je n'avais pas d'équipement ni rien pour apprendre et essayer des choses. C'est alors que j'ai appris ce qu'était un Homelab.

### Qu'est-ce qu'un Homelab ?
Vous derriere, je vous entends ... Qu'est-ce qu'un Homelab ?
> Un homelab, dans les termes les plus simples, est un environnement où vous pouvez apprendre et jouer avec des technologies nouvelles ou peu familières.  Il peut s'agir d'un simple ensemble de machines virtuelles sur un vieux PC ou un ordinateur portable ou d'un réseau complet complexe avec son propre ASN (nous y viendrons, éventuellement...).

### Serveurs privés virtuels avec OVH

#### VPS Numéro 1 : Le site web
Après avoir fait quelques recherches, j'ai conclu deux choses :

1. Le service que je voulais héberger : mon site web.
1. L'emplacement approximatif où je voulais qu'il soit hébergé : La France

Après quelques recherches, j'ai trouvé [OVH](https://ovh.fr), un hébergeur basé en France. OVH proposait et propose toujours des serveurs privés virtuels bon marché et de très bonne qualité. J'ai choisi un serveur 1vCore et 512 Mb RAM pour faire tourner mon site web basé sur Wordpress [^wordpress]. 

Ce VPS [^vps] a fini par héberger le site web d'EvansMedia, ma société de services informatiques et web jusqu'au 10 mars 2021 et ce que je nomme *'Le BBQ d'OVH*' [^ovhfire], où leurs quatre datacenters de Strasbourg sont partis en fumée (littéralement).

[^wordpress]: [Wordpress](https://wordpress.org) est un système de gestion de contenu gratuit et open-source. En 2021, 30% des sites web du monde fonctionnaient avec Wordpress.

[^vps]: Serveur privé virtuel
[^ovhfire]: *10 mars 2021* - Le site de l’entreprise OVH à Strasbourg touché par un « important incendie » - [Le Monde](https://www.lemonde.fr/societe/article/2021/03/10/a-strasbourg-un-important-incendie-sur-le-site-de-l-entreprise-ovh-classe-seveso_6072548_3224.html)

#### VPS numéro 2 : Le serveur de stockage