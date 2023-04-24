---
title: Découverte de Caddy - un serveur web simple et rapide
date: 2023-04-23T09:19:19.413Z
tags:
    - web
    - serveur
    - website
    - caddy
    - apache
    - nginx
    - homelab
summary: Nous avons tous utilisé Apache et NGNIX pour héberger un site web et ils sont très bien, mais pour être honnête, je ne me suis jamais vraiment senti à l'aise avec leur configuration. Et c'est ce qui m'a amené à Caddy. Découvrons ce que Caddy a à offrir.
featuredImage: valery-sysoev-p9OkL4yW3C8-unsplash.webp
---
Nous avons tous utilisé Apache et NGNIX pour héberger un site web et ils sont très bien, mais pour être honnête, je ne me suis jamais vraiment senti à l'aise avec leur configuration. Et c'est ce qui m'a amené à [Caddy](caddyserver.com/). Découvrons ce que [Caddy](caddyserver.com/) a à offrir.

# Caddy
[Caddy](caddyserver.com/) est un serveur web unique _(du moins je le pense)_ avec un jeu de fonctionnalités modernes. Vous pouvez l'utiliser comme [reverse proxy et load balancer.](https://caddyserver.com/docs/proxy) [héberger vos applications PHP avec lui.](https://caddyserver.com/docs/fastcgi) Et même [déployer votre site](https://caddyserver.com/docs/git) avec ``git push``. Et j'ai presque oublié l'une des meilleures fonctionnalités ; le HTTPS automatique, avec des certificats utilisant des fournisseurs comme [Lets Encrypt](https://letsencrypt.org/) et [ZeroSSL](https://zerossl.com/).
 
Cool, non ?

{{<figure src="/img/bbt-give-it-to-me-amy.gif">}}

## Installation

Caddy est disponible sous la forme d'un simple exécutable, d'une [image docker](https://hub.docker.com/_/caddy) ou, comme d'habitude, via un gestionnaire de paquets. Aujourd'hui, nous utiliserons Debian. Mais si vous utilisez un autre système d'exploitation, leur [documentation](https://caddyserver.com/docs/install) devrait vous couvrir.

```bash
sudo apt install -y debian-keyring debian-archive-keyring apt-transport-https
curl -1sLf 'https://dl.cloudsmith.io/public/caddy/stable/gpg.key' | sudo gpg --dearmor -o /usr/share/keyrings/caddy-stable-archive-keyring.gpg
curl -1sLf 'https://dl.cloudsmith.io/public/caddy/stable/debian.deb.txt' | sudo tee /etc/apt/sources.list.d/caddy-stable.list
sudo apt update
sudo apt install caddy
```

Une fois installé, le service s'exécutera automatiquement et servira la page de bienvenue par défaut sur le port 80. Caddy peut ensuite être configuré en utilisant le fichier ```Caddyfile``` dans ```/etc/caddy```

## Configuration

Comme dit plus haut, caddy peut être configuré en utilisant le fichier ```Caddyfile``` écrit en JSON dans ```/etc/caddy```. Dans ce fichier, nous lui donnons des [directives](https://caddyserver.com/docs/caddyfile/directives) pour qu'il soit soit un reverse proxy, un serveur web, un front php, etc. Caddy peut aussi être configuré en utilisant son [API REST](https://caddyserver.com/docs/api) si c'est quelque chose qui vous intéresse.

### Reverse Proxy

Disons que vous avez une application qui fonctionne sur le port 9999 et que vous voulez faire un reverse proxy vers 443 (HTTPS). Le fichier de configuration ressemblerait à ceci :

```json
demo.app.com {
    reverse_proxy 127.0.0.1:9999
    encode gzip
}
```

Pour que Caddy recharge le ```Caddyfile```, il suffit de lancer un ```systemctl reload caddy```, Caddy rechargera alors le fichier de configuration, demandera un certificat pour ```demo.app.com``` et servira votre application sur HTTPS[^HTTPS].

### Fichiers statiques

Ok un reverse proxy c'est bien mais mon site web (celui-ci) est statique et généré par [Hugo](gohugo.io), comment pourrions-nous le servir en utilisant caddy ? Encore une fois, 4 lignes de configuration suffisent.

```json
demo.website.com {
    # où sont stockés nos fichiers statiques
    root * /var/www/demo.droapp.com/public
    file_server 
}
```

Un rapide ```systemctl reload caddy``` plus tard et Caddy sert vos fichiers statiques sur HTTPS[^HTTPS] à ```demo.website.com```.

[^HTTPS]: Caddy ne réussira à demander un certificat que si les ports 80 et 443 sont ouverts sur votre serveur. Vous pouvez également [désactiver la demande de certificat](https://caddyserver.com/docs/automatic-https#activation) si vous n'en avez pas besoin. D'autres [options](https://caddyserver.com/docs/automatic-https) comme le HTTPS local peuvent également être utilisées.

### PHP-fpm

Vous utilisez habituellement Nginx comme façade/front pour PHP-fpm, mais Caddy peut aussi le faire. Et encore une fois avec un simple fichier de configuration.

```json
demo.website.com {
        # où sont stockés nos fichiers
        root * /var/www/demo.website.com
        # on indique à caddy où trouver le socket php-fpm
	    php_fastcgi /run/php/php8.2-fpm.sock	
        file_server
        encode gzip
}