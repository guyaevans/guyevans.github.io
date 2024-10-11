---
title: La publication de mon site avec Hugo, Caddy et Github actions
date: 2024-10-11T15:28:00+02:00
rssFullText: true
lightgallery: true
summary: Je me souviens avoir enregistré mon propre nom de domaine à l'âge de 13 ans et avoir créé mon premier site web avec [Microsoft Frontpage](https://fr.wikipedia.org/wiki/Microsoft_FrontPage), puis plus tard avec [Adobe Dreamweaver](https://en.wikipedia.org/wiki/Adobe_Dreamweaver). Plus tard, j'ai commencé une carrière dans l'informatique et j'ai conçu quelques sites web avec [Wordpress.](https://wordpress.org/) Voyons ce que j'utilise aujourd'hui pour publier ce site web.

---
## Intro
Je me souviens avoir enregistré mon propre nom de domaine à l'âge de 13 ans et avoir créé mon premier site web avec [Microsoft Frontpage](https://fr.wikipedia.org/wiki/Microsoft_FrontPage), puis plus tard avec [Adobe Dreamweaver](https://en.wikipedia.org/wiki/Adobe_Dreamweaver). Plus tard, j'ai commencé une carrière dans l'informatique et j'ai conçu quelques sites web avec [Wordpress.](https://wordpress.org/) Voyons ce que j'utilise aujourd'hui pour publier ce site web.

Aujourd'hui, je me suis éloigné de Wordpress, même s'il s'agit d'une excellente plateforme pour publier un blog et créer des sites web. (Wordpress serait à l'origine de 43 % des sites web en ligne)[^w3] mais mon site web a évolué d'une simple pqge de garde à un endroit où je poste régulièrement des réflexions et du contenu que je trouve intéressant et que j'espère que quelqu'un d'autre trouve intéressant (du moins, j'essaie...).

Je ne veux plus me prendre la tête avec Wordpress, Php, Mysql, les mises a jours des thèmes et des plugins à n'en plus finir. Je veux juste me concentrer sur l'écriture du contenu, et c'est ainsi que la stack technique de mon site web a vu le jour. 

[^w3]: Rapport par [W3 Techs](https://w3techs.com/technologies/overview/content_management)

Qu'est-ce que j'utilise pour créer et publier ce site web ? Prenons un verre et plongeons dans le vif du sujet.

{{<image src="/img/bbt-penny-drinkswebp.webp">}}

## Stack Technique

Je l'ai basé sur trois éléments essentiels :

* [Hugo](https://gohugo.io/) - Un générateur de site statique open-source
* [Caddy](https://caddyserver.com/) - Le serveur web actuel pour héberger le site
* [Github Actions](https://github.com/features/actions) - Un service pour construire les fichiers statiques avec Hugo et les envoyer sur le serveur Caddy.

Jetons un coup d'œil à chaque service.

### Hugo

Si vous cherchez, il existe de nombreux générateurs de sites statiques, comme [Ghost](https://ghost.org/), [Astro](https://astro.build/), [Jeykll](https://jekyllrb.com/) [parmis tant d'autres](https://jamstack.org/generators), chacun utilisant un langage de programmation différent. Je ne me souviens plus très bien comment j'ai trouvé Hugo, mais il s'est démarqué par sa simplicité.

[Hugo](https://gohugo.io/) est un générateur de site statique open-source. Oui, vous avez bien lu, je ne suis pas un developpeur web, donc plus de PHP ici. 

Vous lui donnez un thème, du contenu écrit en [Markdown](https://www.markdownguide.org/) (point bonus) et du HTML si nécessaire. Il supporte aussi les emojis :smile:, les vidéos embarquées, les posts twitter (avant que Mister Elon ne tue cette fonctionnalité). Une fois que votre contenu est prêt, Hugo convertit le tout en HTML statique en quelques secondes, prêt à être uploadé sur votre serveur web.

[Hugo](https://gohugo.io/) facilite également le développement local, en vous permettant de créer une copie locale de votre site web et d'obtenir un aperçu en direct de vos modifications.

Pour commencer, tout ce que vous avez à faire est de [l'installer](https://gohugo.io/getting-started/quick-start/).

### Caddy

[Caddy](https://caddyserver.com/) mon amour ... par où commencer ? Je n'ai jamais réussi à m'y retrouver dans les configurations d'Apache ou de Nginx, mais celle de Caddy est simple.

[Caddy](https://caddyserver.com/) est un serveur web simple, rapide et multiplateforme écrit en Go.J'ai [écrit sur caddy ici](https://guy-evans.com/fr/posts/2023-04-23_a-look-at-caddy-a-simple-and-fast-web-server/) dans le passé lorsqu'il ne servait que certaines parties de ce site web, à savoir le système de commentaires et l'analyse. Mais récemment, j'ai transféré le coeur de mon site web sur caddy car il est tellement simple.

Voici la configuration pour le site web uniquement (pas de commentaires ni d'analyses)

```json {title=Caddyfile}
guy-evans.com www.guy-evans.com {
	root * /srv/guy-evans.com
	encode zstd gzip
	file_server
        log {
                output file /log/guy-evans-fr-caddy_access.log
  }
}
```

Oui, Oui. Huit lignes de configuration pour héberger le site :sunglasses:.

### Github Actions

[Github Actions](https://github.com/features/actions) est une plateforme d'intégration continue et de livraison continue (CI/CD) qui vous permet d'automatiser votre pipeline de construction, de test et de déploiement. 
En gros, cela me permet d'exécuter automatiquement ``hugo`` pour construire les fichiers statiques et ensuite les uploader sur mon serveur web Caddy, le tout à partir d'un simple commit sur le repo Github où je stocke le code source. 
Plus besoin de générer manuellement les fichiers statiques localement sur mon ordinateur portable, puis d'utiliser SFTP pour les télécharger sur le serveur, tout est fait à ma place.

```yaml {open=true}
name: Hugo CI & Deploy
on:
    push:
        branches:
            - master
    pull_request:
    # Permet d'exécuter ce workflow manuellement à partir de l'onglet Actions
    workflow_dispatch:
jobs:
    build:
        name: Build and deploy website
        runs-on: ubuntu-latest
        steps:
        # On fait un checkout sur la branche master avec les sous modules
            - name: Checkout
              uses: actions/checkout@v3
              with:
                  submodules: true
        # On configure Hugo avec la version que nous voulons
            - name: Setup Hugo
              uses: peaceiris/actions-hugo@v2
              with:
                  hugo-version: "latest"
                  extended: true
        # On demande a Hugo de construire les fichiers statiques
            - name: Build website with Hugo
              run: hugo --minify
        # S'il s'agit d'un commit sur la branche master, on utilise Rsync pour envoyer les fichiers au serveur web en utilisant les secrets pour se connecter.
            - name: Deploy website with rsync
              if: github.ref == 'refs/heads/master'
              uses: burnett01/rsync-deployments@5.2
              with:
                  switches: -avzr --quiet --delete
                  path: public/
                  remote_path: ${{ secrets.REMOTE_PATH }}
                  remote_host: ${{ secrets.REMOTE_HOST }}
                  remote_user: ${{ secrets.REMOTE_USER }}
                  remote_key: ${{ secrets.REMOTE_KEY }}
                  remote_port: ${{ secrets.REMOTE_PORT }}
```

A chaque fois que j'effectue un commit sur la branche master de mon repo, le workflow 
est exécuté

## Fin
Comme vous le voyez, avec cet ensemble de technologies, je peux me concentrer sur l'écriture de contenu et ne pas me préoccuper de la technologie derrière mon site web. J'ai maintenant un flux de travail simple et rapide pour publier du contenu. Pas de PHP, pas de MYSQL, pas de plugins à se soucier. Comme d'habitude, tout est mieux avec la philosophie KISS[^kiss].

[^kiss]: Keep it simple, stupid