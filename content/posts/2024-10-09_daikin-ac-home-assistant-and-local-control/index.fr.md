---
title: Clim Daikin, Home Assistant et contrôle local
date: 2024-10-09T21:23:43+02:00
tags:
  - Home Assistant
  - IoT
  - AC
rssFullText: true
featuredImage: feature.jpg
lightgallery: true
summary: Lorsque nous avons acheté notre appartement, les anciens propriétaires avaient installé un mini-split AC Daikin dans le salon. Super, encore une chose à intégrer avec Home Assistant. Mais les modules Wifi de Daikin sont beaucoup trop chers et dépendent du cloud.
---
## Climatisation et Home Assistant
Lorsque nous avons acheté notre appartement, les anciens propriétaires avaient installé un mini-split AC Daikin dans le salon. Super, encore une chose à intégrer avec Home Assistant.[^HA].


[^HA]: [Home Assistant](https://www.home-assistant.io/) - Logiciel domotique open source qui privilégie le contrôle local et le respect de la vie privée. Développée par une communauté mondiale de bricoleurs et de passionnés de bricolage

J'ai d'abord fait ce que j'ai toujours fait dans mes précédents appartements et j'ai utilisé un [Broadlink Mini](https://amzn.to/4eAfGEb) et l'intégration [SmartIR](https://github.com/smartHomeHub/SmartIR/) pour permettre un contrôle intelligent du minisplit à l'aide de Home Assistant. Le contrôle par infrarouge est très bien mais manque de certaines fonctionnalités, principalement le feadback. Si votre appareil ne reçoit pas la commande IR, eh bien, ils sont maintenant désynchronisés et vous n'en savez rien, et votre climatiseur ne s'est pas allumé ...


## Modules Daikin et une trouvaille

Daikin fabrique un [module wifi](https://amzn.to/47VGXyB) à ajouter au mini split, mais il présente deux défauts majeurs :

* le prix : Je les ai trouvés pour 50-90 Euros *(en fonction de la version dont vous avez besoin)* - beaucoup trop cher ...
* contrôle dans le cloud uniquement : L'application Daikin permettait un contrôle local, mais après quelques mises à jour, elle est passée à un contrôle en cloud uniqument, ce qui signifie que sans connexion Internet, vous pouvez dire bye bye à votre climatiseur ...

Lors de mes recherches sur le module Daikin, je suis tombé sur ce post sur le forum Home Assisant : [Need Daikin Wifi? Use the Open-Source Faikin ESP32 Hardware instead of the official wifi Modules](https://community.home-assistant.io/t/need-daikin-wifi-use-the-open-source-faikin-esp32-hardware-instead-of-the-official-wifi-modules/644370) *(Anglais)* et ce repo github [ESP32-Faikin](https://github.com/revk/ESP32-Faikin) par "RevK'.

J'ai décidé d'en acheter un ainsi que des fils de raccordement, après tout ... 

## Le matériel

{{<image caption="Comment cela peut-il être difficile ?" src="/img/clarkson-how-hard.gif">}}

J'ai reçu le PCB et les fils de raccordement assez rapidement, merci Amazon

C'est maintenant que vient la partie la plus amusante, celle où l'on raccorde le tout. *Mais d'abord un avertissement* :zap: 🚨:

{{< admonition type=danger title="L'électricité est dangereux et peut vous tuer">}}
Coupez votre disjoncteur principal avant de démonter votre unité de climatisation, si vous n'êtes pas sûr, appelez un professionnel.
*Ne dites pas que je ne vous ai pas prévenu si vous recevez une décharge électrique ou si vous cassez quelque chose.*{{< /admonition >}}

Les mini-splits se démontent généralement assez facilement. Une ou deux vis et quelques clips maintiennent le châssis.

{{<image src="/posts/2024-10-09_daikin-ac-home-assistant-and-local-control/naked_split.jpg" class="center" width="90%" caption="Et regardez un minisplit tout nu">}}

Une fois le capot de l'AC enlevé, il faut chercher la carte mère. Elle est généralement située sur le côté. Et pour ma clim, la prise que nous devons utiliser est étiquetée S21. **Ne touchez à rien d'autre ici. Surtout les condensateurs.** :zap:

{{<image src="/posts/2024-10-09_daikin-ac-home-assistant-and-local-control/pcb_circuit.jpg" class="center" width="90%" caption="La carte mère de la clim et le circuit imprimé Faikin installé - **Ne touchez pas à ce condensateur**">}}

Une fois l'installation terminée, nous pouvons remettre le couvercle métallique en place. Et il y a un espace parfait pour le carte Faikin en dessous.

{{<image src="/posts/2024-10-09_daikin-ac-home-assistant-and-local-control/installed_pcb.jpg" class="center" width="90%" caption="Installation finale du PCB">}}

Après avoir remis tous les couvercles sur l'unité de climatisation, nous pouvons rallumer notre disjoncteur principal.

*Vous l'avez éteint, j'éspère ?* :eyes:

Une fois que votre climatiseur sera à nouveau alimenté, vous pourrez vous connecter au Faikin via le wifi.

{{<image src="https://github.com/revk/ESP32-Faikin/blob/main/Manuals/WiFi1.png?raw=true" width="90%" class="center" caption="Choisir le wifi diffusé par la carte PCB">}}

Il vous demandera ensuite d'entrer les détails de votre réseau wifi et de votre brokeur MQTT.

{{<image src="https://github.com/revk/ESP32-Faikin/blob/main/Manuals/WiFi3.png?raw=true" width="90%" caption="Entrez vos coordonnées wifi et MQTT">}}

Une fois cela fait, vous serez face à des commandes de climatisation et devinez quoi? Elles fonctionnent instantanément.

{{<image src="https://github.com/revk/ESP32-Faikin/blob/main/Manuals/Controls.png?raw=true" width="90%" caption="Des commandes de climatisation">}}

Enfin, nous devons configurer le côté Home Assistant.

## Home Assistant

Il n'y a pas grand chose à faire ici. Si votre broker MQTT est correctement configuré dans Home Assistant. Votre unité de climatisation devrait s'afficher d'elle-même.

{{<image src="/posts/2024-10-09_daikin-ac-home-assistant-and-local-control/ha1.png" class="center" width="90%" caption="Unité climatisation dans Home Assistant">}}

{{<image src="/posts/2024-10-09_daikin-ac-home-assistant-and-local-control/ha2.png" class="center" width="90%" caption="Commandes de climatisation dans Home Assistant">}}

Avec cela, nous avons maintenant un contrôle **local** complet sur notre unité de climatisation qui comprend un retour d'information sur l'état de la climatisation à Home Assistant.

Plus besoin de se demander pourquoi il fait chaud dans la pièce et pourquoi la climatisation ne s'est pas allumée.

---
*Note : certains liens peuvent être des liens d'affiliation. En tant qu'associé d'Amazon, je peux percevoir une petite commission sur les achats qualifiés, sans frais supplémentaires pour l'acheteur.*