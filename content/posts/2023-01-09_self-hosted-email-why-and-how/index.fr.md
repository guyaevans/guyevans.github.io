---
title: Mails auto-hébergé - Pourquoi et comment?
date: 2023-01-15T14:00:58.009Z
featuredImage: brett-jordan-lpzy4da9aro-unsplash.jpg
summary: "Question : Quel est le service sur Internet que nous utilisons régulièrement et qui fait partie de notre quotidien ? **L'EMAIL**

Il y a de fortes chances que si vous utilisez un ordinateur, vous ayez au moins une adresse e-mail, certains en ont plus d'une (j'ai perdu le compte). Pourquoi devriez-vous avoir le vôtre ?"
tags:
  - email
  - self-hosted
  - mailcow
  - guide
rssFullText: true
authors: ["Guy Evans"]
---
## Email
Question : Quel est le service sur Internet que nous utilisons régulièrement et qui fait partie de notre quotidien ? **L'EMAIL**

Il y a de fortes chances que si vous utilisez un ordinateur, vous ayez au moins une adresse e-mail, certains en ont plus d'une (j'ai perdu le compte). Cela m'amène à une conviction profonde : si vous travaillez dans l'informatique sous une forme ou une autre, votre adresse électronique devrait être la vôtre, avec votre propre nom de domaine _(hello@evans.fr par exemple)_ au lieu d'utiliser un @gmail.com @outlook.com ou @aol.com _(c'est vieux ça) _ ou tout autre fournisseur.

## Pourquoi (l'auto-hébergement)
Pour ma part, j'avais l'habitude d'héberger toutes mes adresses e-mail sur Google Apps _(maintenant appelé Google Workspace)_. Il était simple de faire pointer les bons enregistrements mx vers leurs serveurs et voila votre propre messagerie.

De nombreux autres fournisseurs vous permettent également de faire cela, mais là où Google a été intelligent, c'est que leur service avait un excellent niveau gratuit (comme la bière gratuite) jusqu'à ce qu'ils décident de supprimer le niveau gratuit. (Une autre raison pour laquelle les gens peuvent héberger eux-mêmes leurs e-mails est d'avoir un contrôle total sur le service et de ne pas partager de données avec les GAFAM).

Cela m'a fait penser, pourquoi ne pas héberger mon propre email un jour. Donc, comme pour beaucoup de mes recherches, j'ai commencer sur Reddit.com et le consensus général est : Pourquoi s'embêter. Mais je continue quand même mes recherches et décide de continuer mon projet. Après tout, la meilleure façon d'apprendre est de le faire. Comment cela peut-il être aussi difficile ?

_mais d'abord, il nous faut du café..._
{{<figure src="/img/bbt-first-coffee.gif">}}

## Comment (en utilisant Docker et Mailcow)

Je ne vais pas vous expliquer en détail comment configurer un serveur de messagerie, car il existe de nombreux guides à ce sujet, mais je vais au moins vous mettre sur la bonne voie.

### Ce dont nous avons besoin

1. Un nom de domaine de votre choix
2. Un serveur VM ou VPS avec : 2 VCPU, 4 Go de RAM (au minimum), 80 Go de stockage (ou plus), une adresse IP propre. _(Je suggère [Milkywan](https://milkywan.fr/) ou [Virtua.Cloud](https://www.virtua.cloud/), tous deux basés en France et disposant d'une excellente connectivité)._
3. Docker & Docker Compose (je suggère d'utiliser Docker car il est plus rapide à mettre en place).
4. [Mailcow Dockerized](https://mailcow.email/), mon serveur de messagerie préféré - il contient tous les services nécessaires et fonctionne avec des conteneurs Docker.

### Configuration
#### DNS
Pour que Mailcow fonctionne, vous devez configurer les enregistrements DNS suivants

```
# Name              Type       Value
mail                IN A       1.2.3.4
autodiscover        IN CNAME   mail.example.org. (your ${MAILCOW_HOSTNAME})
autoconfig          IN CNAME   mail.example.org. (your ${MAILCOW_HOSTNAME})
@                   IN MX 10   mail.example.org. (your ${MAILCOW_HOSTNAME})
```

#### Installation de Mailcow

Une fois que les enregistrements DNS sont propagés, vous pouvez installer et configurer Mailcow.
Tout d'abord, nous clonons le référentiel et définissons les permissions sur les répertoires.
```bash
su
umask
0022 # <- Vérifier que c'est bien 0022
cd /opt
git clone https://github.com/mailcow/mailcow-dockerized
cd mailcow-dockerized
```
Une fois que vous aurez répondu aux questions, le fichier de configuration sera généré pour vous. Une fois généré, nous pouvons lancer mailcow en utilisant les commandes suivantes
```bash
docker compose pull
docker compose up -d
```
FÉLICITATIONS ! Vous avez installé un serveur de messagerie (enfin presque).

Maintenant que le serveur fonctionne, vous pouvez accéder à https://${MAILCOW_HOSTNAME} avec les informations d'identification par défaut ```admin``` + le mot de passe ```moohoo``` et commencer à configurer les domaines et les adresses e-mail.

Mailcow a un excellent [site d'aide](https://docs.mailcow.email/post_installation/firststeps-ssl/) qui vous guidera à propos de la plupart des paramètres que je n'ai pas couverts.

## Fin
J'espère qu'avec ce court article, vous pouvez voir qu'il n'est pas vraiment difficile de mettre en place un serveur de messagerie. Ce qui est difficile, c'est de garder ses addrresses hors des listes de spam et de combattre le spam entrant, mais pour un usage personnel et avec l'aide de Mailcow, vous devriez vous en sortir.

Une autre chose que vous ne devez pas oublier est ... LES SAUVEGARDES. 

Si vous hébergez votre courriel principal sur ce serveur, vous devriez le sauvegarder régulièrement.