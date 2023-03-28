---
title: Sauvegardes avec Restic
date: 2023-03-28T16:17:14.251Z
featuredImage: immo-wegmann-htsvneqf5fg-unsplash.jpg
tags:
  - backups
  - restic
  - self-hosted
summary: Après avoir eu la chance de ne perdre aucune de mes données
  sauvegardées sur mon nextcloud après l'incendie d'OVH, j'ai commencé à
  chercher une solution de sauvegarde décente et j'ai trouvé Restic. Voyons
  comment cela fonctionne.
---
Après avoir eu la chance de ne perdre aucune de mes données sauvegardées sur mon nextcloud suite à l'incendie d'OVH[^ovhfire] :fire: à Strasbourg. J'ai cependant perdu le VPS qui hébergeait le site web de [www.evansmedia.fr](https://www.evansedia.fr)

[^ovhfire]: *10 mars 2021* - Le site de l’entreprise OVH à Strasbourg touché par un « important incendie » - [Le Monde](https://www.lemonde.fr/societe/article/2021/03/10/)

> Les cordonniers sont toujours les plus mal chaussés, n'est-ce pas ?
>
> {{<figure src="/img/this-is-fine.webp">}}

Je suis peut-être devenu un peu paranoïaque en ce qui concerne les sauvegardes, en particulier pour tout ce qui est stocké dans mon Nextcloud, qui contient à peu près tout ce qui a trait à ma vie personnelle, y compris les photos de notre nouveau-né.

## The 3-2-1 Rule
J'ai décidé de bâtir ma stratégie de sauvegarde sur la règle des 3-2-1 (et vous devriez en faire autant)

L'idée qu'une solution de sauvegarde minimale devrait comprendre trois copies des données, dont deux copies sur des supports différents et une copie distante (hors site).

 {{<figure src="./3-2-1-Backup-Rule.png">}}

### Sauvegardes Locales

*(Nous simulons ici la sauvegarde d'un serveur local, mais la machine virtuelle que je sauvegarde est en fait hébergée dans un centre de données, je la sauvegarde donc localement à la maison)*

Pour effectuer mes sauvegardes, j'utilise deux outils, Proxmox Backup Server[^pbs] qui est une application de sauvegarde construite par les créateurs de proxmox ; et Restic, une application moderne qui sauvegarde des fichiers vers des tas de services différents. Aujourd'hui, nous allons nous concentrer sur un seul niveau de sauvegarde avec Restic.

[^pbs]: [Proxmox Backup Server](https://www.proxmox.com/en/proxmox-backup-server) est une solution de sauvegarde qui permet de sauvegarder et de restaurer des VM, des conteneurs et des hôtes physiques.

## Restic

> restic est un logiciel de sauvegarde rapide, efficace et sûr. Il supporte les trois principaux systèmes d'exploitation (Linux, macOS, Windows) et quelques autres plus petits (FreeBSD, OpenBSD).
> [https://restic.net/](https://restic.net/)

### Pourquoi Restic
- il est simple et rapide à utiliser
- il permet de sauvegarder de nombreux emplacements de stockage différents, y compris des services auto-hébergés et en ligne (Amazon S3, Google Drive, Backblaze, etc.)
- ses sauvegardes peuvent être entièrement __chiffrées__
- les restaurations sont simples et vous pouvez même monter le dépôt de sauvegarde et récupérer un seul fichier.

### Installer Restic

Tout d'abord, nous devons installer Restic, c'est aussi simple que d'utiliser APT sur Ubuntu/Debian (dans mon cas d'utilisation).

```bash
sudo apt install restic
```

### Initialiser le dépôt de stockage
La beauté de Restic est que vous pouvez utiliser à peu près n'importe quel type de stockage (local, Amazon S3, Minio Server, Backblaze B2, Azure, Google Cloud Storage, SFTP, etc.) pour votre repo (dépot) de sauvegarde et l'interface reste en grande partie la même. Dans ce cas, j'utilise leur propre backend de stockage. [__Rest-Server__](https://github.com/restic/rest-server)

{{< admonition info "Rest-Server">}}
Rest Server est un serveur HTTP haute performance qui met en œuvre l'API REST de restic. Il fournit un moyen sûr et efficace de sauvegarder des données à distance, en utilisant le client de sauvegarde restic via l'API rest : URL.
{{< /admonition >}}

Maintenant que vous avez choisi le backend de stockage pour votre dépôt, nous devons l'initialiser.

Disons que nous voulons nommer les sauvegardes de notre dépôt et nous pouvons utiliser la commande suivante pour l'initialiser.
```bash
restic -r rest:http://my-rest-server:8000/backups init
```

Restic vous demandera de créer un mot de passe (gardez-le précieusement, nous en aurons besoin plus tard) et initialisera le dépôt. Nous pouvons utiliser ``check`` pour vérifier que le dépôt est initialisé.

```bash
restic -r rest:http://my-rest-server:8000/backups check
enter password for repository:
password is correct
create exclusive lock for repository
load indexesY
check all packs
check snapshots, trees and blobs
no errors were found
```

### La Sauvegarde

Maintenant que nous avons un dépôt initialisé, nous pouvons commencer à sauvegarder nos photos de chats. Avec la commande ```backup``` et pointant vers le dépôt que nous avons créé. Ainsi, pour sauvegarder mon dossier ```~/cat-pictures```, je lancerais la commande suivante
```bash
restic -r rest:http://my-rest-server:8000/backups backup ~/cat-pictures
```

Oui, c'est aussi simple que ça :sunglasses:

A chaque fois qu'une sauvegarde est lancée, restic crée un snapshot, vous pouvez les lister en utilisant la commande ```snapshot```.

```bash
restic -r rest:http://my-rest-server:8000/snapshots
enter password for repository:
password is correct
ID        Date                 Host        Tags        Directory
----------------------------------------------------------------------
ac8721ec  2023-03-26 20:18:16  my-pc              /home/bounty/cat-pictures
----------------------------------------------------------------------
1 snapshots
```


### Restauration des photos de chats

Qu'est-ce qu'une sauvegarde si vous n'avez pas testé une restauration ? Ce n'est pas une sauvegarde parce que vous ne savez pas si elle a fonctionné.

Essayons donc de restaurer nos images de chats avec la commande ```restore```. Nous devons spécifier l'identifiant du snapshot ```ac8721ec``` et l'endroit où nous voulons restaurer ```~/restored-cat-pictures```

```bash
restic -r rest:http://my-rest-server:8000/backups restore ac8721ec --target ~/restored-cat-pictures/
```

{{< admonition tip>}}
Vous pouvez utiliser ``latest`` à la place de l'identifiant du snapshot pour référencer le dernier snapshot.{{</ admonition>}}

C'est aussi simple que la sauvegarde, non ?
{{< admonition info "Autres commandes et documentation">}}
Restic possède de nombreuses autres commandes utiles que vous pouvez trouver dans leur excellente [documentation] (https://restic.readthedocs.io).
{{</ admonition>}}

### Automatisation avec un script Bash et Cron

Les sauvegardes manuelles sont bien mais vous risquez d'oublier de les exécuter, nous allons donc automatiser notre sauvegarde avec un script bash et une entrée cron. Et pour les points bonus, assurons-nous d'avoir un moyen de nous envoyer un message en cas d'échec. Mon script de base est ci-dessous, chaque action est expliquée dans les commentaires.

_Note : mon script est loin d'être parfait et peut contenir des erreurs, n'hésitez pas à le corriger ou à suggérer des améliorations ci-dessous_

```bash
#!/bin/bash
#This will run Restic backups

# Variables Restic
export RESTIC_REPOSITORY=rest:http://my-rest-server:8000/backups
export RESTIC_PASSWORD=super-secret-repo-password1234
RESTIC_Folder="~/cat-pictures"
# On choisi ou on ecrit les logs
Log_location=/var/log/restic-nas.log
# Variables Telegram (si elles ne sont pas utilisées, vous devez supprimer le bloc entier pour telegram)
TELEGRAM_token=
TELEGRAM_chatid= 

#Defini une fonction d'horodatage
timestamp() {
date "+%b %d %Y %T %Z"
}

# Inserer l'horodatage et l'en-tête dans le journal
printf "\n\n"
echo "-------------------------------------------------------------------------------" | tee -a $Log_location
echo "$(timestamp): restic.sh - Rest Backup started" | tee -a $Log_location

# Executer des sauvegardes
NAS_backup_log=$(restic backup $RESTIC_Folder)
echo "$NAS_backup_log" | tee -a $Log_location
## Il s'agit d'un moyen simple de vérifier que la sauvegarde a été un succès et que les fichiers ont été ajoutés à la base de données, sinon nous échouons.
if [[ "$NAS_backup_log" == *"Added to the repo"** ]]; then
## Si la sauvegarde a été réussie, nous pouvons continuer et écrire le résultat dans le journal.
	printf "\n\n"
	echo "-------------------------------------------------------------------------------" | tee -a $Log_location
	echo "$(timestamp): restic.sh - COMPLETED - Rest Backup" | tee -a $Log_location
else
## Si la sauvegarde échoue, nous écrivons dans le journal et envoyons un message sur Telegram.
	printf "\n\n"
	echo "-------------------------------------------------------------------------------" | tee -a $Log_location
    echo "$(timestamp): restic.sh - FAILED - Rest Backup" | tee -a $Log_location
## envoyer un message à telegram
	curl -X POST https://api.telegram.org/$TELEGRAM_token/sendMessage \
	       	--data parse_mode=HTML --data chat_id="$TELEGRAM_chatid" \
		--data text="<b>$(timestamp)%0A FAILED - REST Backup</b>%0A%0A<pre>$NAS_backup_log</pre>" -4
	exit 1
fi
```

Une fois que nous avons sauvegardé notre script bash sous ```~/restic_script.sh```, nous devons autoriser son exécution avec ```chmod``` et l'ajouter à notre [crontab](https://fr.wikipedia.org/wiki/Cron).

```bash
chmod +x ~/restic_script.sh # permettre l'exécution de notre script
```

```bash
crontab -e # éditer notre crontab
```
```bash
# Edit this file to introduce tasks to be run by cron.
[...]
# To define the time you can provide concrete values for
# minute (m), hour (h), day of month (dom), month (mon),
# and day of week (dow) or use '*' in these fields (for 'any').
[...]
# m h  dom mon dow   command
0 1 * * * ~/restic_script.sh # exécutera notre script tous les soirs à 1 heure du matin
```
Vous avez maintenant automatisé vos sauvegardes. 

Je suis sûr que vous avez une meilleure façon de procéder, pourquoi ne pas partager vos idées dans les commentaires ?
