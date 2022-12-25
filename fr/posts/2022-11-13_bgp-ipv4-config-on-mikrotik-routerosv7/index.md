# Config BGP IPv4 sur Mikrotik RouterOSv7

Je connais quelques personnes qui font tourner leurs ASN sur des routeurs Mikrotik (sous RouterOS) car leur matériel est bon marché et facile à trouver. Moi y compris, je fait du BGP sur leur image CHR pour annoncer mon propre ASN et mes préfixes IP v4 et V6. RouterOS CHR est une image de RouterOS que vous pouvez exécuter sur votre propre matériel.

Avant on utilisait plus RouterOS version 6, mais maintenant la version 7 a quelques versions stables et est supposée être plus stable et plus rapide pour faire du BGP, j'ai pensé que je partagerais quelques configurations pour la version 7 car la syntaxe de la configuration a beaucoup changé. Aujourd'hui je vais me concentrer sur IPv4 mais IPv6 est à peu près le même.

### Attendez ! ASN, BGP, vous m'avez perdu ...

#### Qu'est-ce que BGP ?
> Border Gateway Protocol (BGP) est le service postal d'Internet. Lorsque quelqu'un dépose une lettre dans une boîte aux lettres, le service postal traite ce courrier et choisit un itinéraire rapide et efficace pour livrer cette lettre à son destinataire[^cloudflarebgp].

[^cloudflarebgp]: Source (Anglais): Qu'est-ce que BGP ? | Le routage BGP expliqué - [Cloudflare](https://www.cloudflare.com/learning/security/glossary/what-is-bgp/)

#### Qu'est-ce qu'un système autonome et les ASN ?
>L'Internet est un réseau de réseaux*, et les systèmes autonomes sont les grands réseaux qui composent l'Internet. Plus précisément, un système autonome (AS) est un grand réseau ou groupe de réseaux qui possède une politique de routage unifiée. Chaque ordinateur ou dispositif qui se connecte à l'Internet est connecté à un AS. 

> Chaque AS se voit attribuer un numéro officiel, ou numéro de système autonome (ASN), de la même manière que chaque entreprise possède une licence commerciale avec un numéro unique et officiel. Mais contrairement aux entreprises, les parties externes font souvent référence aux AS par leur seul numéro.
[^cloudflareasn]

[^cloudflareasn]: Source (Anglais): Qu'est-ce qu'un système autonome ? | Que sont les ASN ? - [Cloudflare](https://www.cloudflare.com/learning/network-layer/what-is-an-autonomous-system/)

![C'est joli ça gif](https://media4.giphy.com/media/U09CkmV0ewDbfGgAZR/giphy.gif?cid=790b7611acdfb9df5539d54ed990cfdf2d9c2469aaf1eee6&rid=giphy.gif&ct=g)

### A quoi ressemble la configuration ?
Voici les exports de configuration pour annoncer mes préfixes via l'AS 211486. Comme vous pouvez le voir, avant de faire quoi que ce soit, vous devez ajouter les préfixes que vous voulez annoncer à une liste de réseaux que vous spécifierez dans le modèle bgp. Le reste devrait être assez explicite mais j'ai détaillé les points principaux ci-dessous.

#### Modèles/Templates
ROSv7 a également introduit le concept de modèles dans BGP, par exemple vous pouvez définir un modèle pour iBGP, un pour les IXP et un autre pour eBGP. La je n'utilise q'un seul

```sh
/routing bgp template
add as=211486 disabled=no input.filter=bgp-in name=as211486-public-out-ip4 output.filter-chain=bgp-out .network=bgp \
    routing-table=main
add address-families=ipv6 as=211486 disabled=no name=as211486-public-out-ip6 output.network=bgp routing-table=main
```
#### Filtres
Comme dans toute configuration BGP, nous avons des filtres. Mikrotik a modifié la syntaxe des filtres dans ROSv7, cela ressemble un peu à bird. Leur [référence](https://help.mikrotik.com/docs/display/ROS/Route+Selection+and+Filters#RouteSelectionandFilters-FilterSyntax) est assez bonne.
Voici un ensemble de base de filtres entrants et sortants. Les deux premières lignes autorisent toutes les routes réseau sous 0.0.0.0/0 mais pas 185.133.194.0/24 qui est le mien. La troisième ligne permet à mon préfixe d'être annoncé et rien d'autre. Si aucun filtre n'est défini pour une route, elle sera rejetée. 

```sh
/routing filter rule
add chain=bgp-in comment="STD IPv4 IN Filter" disabled=no rule="if (dst in 0.0.0.0/0) {set bgp-local-pref 90 ; accept}"
add chain=bgp-in comment="STD IPv4 IN Filter" disabled=no rule="if (dst in 185.133.194.0/24) {reject}"
add chain=bgp-out comment="STD IPv4 OUT Filter" disabled=no rule="if ( dst == 185.133.194.0/24 ) {accept}"
```

#### Connexion
Enfin, nous ajoutons la connexion qui référence le modèle, la liste de réseaux et les filtres que nous venons de créer.

```sh
/routing bgp connexion
add as=211486 disabled=no input.filter=bgp-in local.role=ebgp name=ip4-mw output.filter-chain=bgp-out .network=bgp ``
    remote.address=80.67.167.166/32 router-id=185.133.194.1 routing-table=main templates=as211486-public-out-ip4
```

#### Route Blackhole

Une chose que j'oublie souvent est d'ajouter une route blackhole à ma table de routage pour les préfixes que je voudrais annoncer. Sans cela, vous pouvez avoir les bons filtres mais rien ne sera annoncé car le routeur n'annoncera que les préfixes qui se trouvent dans la table de routage.

```sh
/ip route
add blackhole disabled=no dst-address=185.133.194.0/24
```

Félicitations ! Vous annoncez maintenant un réseau avec BGP sur un RouterOSv7 de Mikrotik.

![félicitations](https://i.giphy.com/media/D1wxiYaBi6euR8W0VX/giphy.webp)
