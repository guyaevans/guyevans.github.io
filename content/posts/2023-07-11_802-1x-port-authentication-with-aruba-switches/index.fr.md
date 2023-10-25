---
title: Authentification des ports 802.1x avec des switches ARUBA
date: 2023-07-13T14:32:51.515Z
featuredImage: thomas-jensen-ISG-rUel0Uw-unsplash.jpg
summary: Imaginez une situation où vous avez une salle de réunion publique
  équipée de ports Ethernet accessibles, et vous voulez assurer un accès
  sécurisé à ces ports. L'une des options est l'authentification de port 802.1X
  - Jetons un coup d'œil à la configuration du côté du switch.
tags:
  - 802.1X
  - Radius
  - Security
  - Switch
  - Aruba
  - Vlan
rssFullText: true
---
## 802.1x - Pourquoi ?

Dans les entreprises, nous utilisons souvent 802.1x pour sécuriser l'accès aux réseaux câblés et sans fil. Pour les réseaux câblés, la norme 802.1x nous permet d'exiger une authentification au niveau du port sur un switch compatible, ce qui nous permet également d'attribuer dynamiquement des VLAN au port et à l'appareil de connexion.

Imaginez une situation dans laquelle vous disposez d'une salle de réunion publique équipée de ports Ethernet accessibles et où vous souhaitez garantir un accès sécurisé à ces ports. Dans un tel scénario, deux options s'offrent à vous :

1. La première option consiste à désactiver tous les ports inutilisés. Bien que cette approche offre un certain niveau de sécurité, elle présente un inconvénient. Que se passe-t-il si quelqu'un a besoin d'utiliser le port ? Dans ce cas, vous devrez vous connecter manuellement au switch, activer temporairement le port, puis le désactiver à nouveau une fois qu'il n'est plus utilisé. Ce processus peut être assez lourd et faire perdre du temps au personnel informatique.

1. La deuxième option, plus efficace, consiste à mettre en œuvre la norme 802.1X. Cette norme offre une solution supérieure en maintenant les ports actifs à tout moment. Avec la norme 802.1X, lorsqu'un appareil de l'entreprise est branché sur le port, il accède automatiquement au réseau de l'entreprise sur le VLAN attribué. En revanche, si un appareil inconnu est connecté, il sera affecté à un VLAN de votre choix, généralement un réseau public avec accès à l'internet uniquement. Cette séparation garantit que les appareils non autorisés ne peuvent pas accéder au réseau de l'entreprise. L'avantage est que vous n'avez plus besoin d'activer et de désactiver manuellement les ports chaque fois qu'un accès est nécessaire.

## De quoi avons-nous besoin ?

* Un serveur RADIUS (à l'IP 192.168.10.2) - nous utilisons Microsoft NPS (je ne détaillerai pas la configuration de NPS ici mais je le ferai peut-être dans le futur).
* Un switch géré - Il peut s'agir de n'importe quel switch géré capable de faire de l'authentification de port, je fais cela sur un [ARUBA 2930F](https://www.arubanetworks.com/products/switches/access/2930f-series/) (qui est un bon switch d'ailleurs, si seulement je pouvais le ramener chez moi :wink: :wink: )
* Quelques VLAN
  * VLAN d'entreprise : 10
  * VLAN public : 20
* Un ordinateur Windows

## Commençons à configurer
### Configuration du switch
#### Configuration des VLAN

La première chose à faire est d'ajouter les vlans si ce n'est pas déjà fait. J'ai taggé les vlans sur tous les ports pour des raisons de simplicité.

```
vlan 10 name VLAN10_Enterprise
vlan 10 tagged all
vlan 20 nom VLAN20_Publique
vlan 20 tagged all
vlan 30 name VLAN30_Telephonie
vlan 30 tagged all
```

#### Serveur Radius et authentification 
Ensuite, nous devons configurer le serveur RADIUS distant (notre serveur NPS) et indiquer au switch la "secretkey"

```
radius-server host 192.168.10.2 key "secretkey"
```

Ensuite, nous configurons EAP-RADIUS, ce qui permet au commutateur de transmettre les paquets d'authentification des clients au serveur Radius.

```
aaa authentication port-access eap-radius
```

Nous activons ensuite 802.1x sur les ports de notre commutateur, ici nous le faisons sur une série de ports mais vous pouvez spécifier des ports individuels, ou les séparer par une virgule. Nous demandons au switch de placer les clients autorisés sur le vlan 10 (auth-vid) et les clients non autorisés sur le vlan 20 (unauth-vid).

```
aaa port-access authenticator 1-10
aaa port-access authenticator 1-10 unauth-vid 20
aaa port-access authenticator 1-10 auth-vid 10
aaa port-access authenticator active
```

La configuration de notre switch est ainsi terminée. Ou bien est-ce le cas ?

#### (Bonus) Qu'en est-il des téléphones IP ?

Vous avez peut-être remarqué que j'ai inclus un vlan pour les téléphones, car c'était une question importante que je me posais lors de mon premier déploiement de 802.1x. Devrai-je ajouter chaque téléphone à notre serveur Radius et configurer chaque téléphone pour l'authentification ou utiliser MAB[^MAB]?

[^MAB]: MAC Authentication Bypass - [MAB - networklessons.com](https://networklessons.com/cisco/ccie-routing-switching-written/mac-authentication-bypass-mab)

Non ! Au lieu de cela, nous allons faire du vlan un vlan voice avec la commande suivante, ce qui permettra au téléphone d'utiliser LLDP pour récupérer le vlan qui lui a été assigné, sans avoir besoin de l'authentification.

```
vlan 30 voice
```

#### Conclusion

La norme 802.1X câblée offre une authentification et un contrôle d'accès au niveau du port, garantissant que seuls les appareils autorisés peuvent se connecter au réseau. Elle permet l'attribution dynamique de VLAN et élimine la nécessité d'une gestion manuelle des ports, ce qui renforce la sécurité du réseau.

Dans cet article de blog, j'ai voulu mettre en évidence la configuration côté commutateur de 802.1X, en me concentrant sur le processus de configuration. Lorsqu'un client non autorisé est connecté au port du switch, le système l'affecte automatiquement au VLAN 10. De même, lorsqu'un téléphone est branché, il rejoint le VLAN 30 sans aucune configuration supplémentaire. En ce qui concerne les clients autorisés, ils sont automatiquement placés dans le VLAN 10.

En résumé, la norme 802.1X câblée offre une authentification et un contrôle d'accès au niveau du port, garantissant que seuls les appareils autorisés peuvent se connecter au réseau. Elle permet l'attribution dynamique de VLAN et élimine la nécessité d'une gestion manuelle des ports, ce qui renforce la sécurité du réseau.

---
_Photo par Thomas Jensen @ [Unsplash.com](https://unsplash.com/fr/photos/ISG-rUel0Uw?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText)_