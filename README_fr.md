🇫🇷 [Version française](README.fr.md) | 🇬🇧 [English version](README.en.md)
# Testeur de ligne-de-vue LoRa multi-nœuds

**Hāmani Fenua · TERRA FORMA**  
Firmware pour **ESP32 / Heltec WiFi LoRa 32 V2** et cartes LoRa compatibles

![Plateforme](https://img.shields.io/badge/plateforme-ESP32-blue)
![Radio](https://img.shields.io/badge/radio-LoRa-green)
![Interface](https://img.shields.io/badge/interface-web%20UI-orange)
![Stockage](https://img.shields.io/badge/stockage-LittleFS-lightgrey)
![Licence](https://img.shields.io/badge/licence-custom-informational)

[English version](README.en.md)

## Présentation

Ce projet transforme une carte **ESP32 + LoRa** en **testeur terrain multi-nœuds de visibilité radio**.

Chaque nœud :

- crée son propre **point d’accès Wi-Fi**
- sert une **interface web locale** sur `192.168.4.1`
- transmet des balises LoRa
- écoute les paquets envoyés par les autres nœuds
- calcule des indicateurs radio utiles
- enregistre les paquets reçus dans des fichiers **CSV** dans **LittleFS**

Le système est destiné aux **tests de ligne de vue**, à la **préparation de déploiements**, à la **comparaison de sites** et à la **validation terrain de liaisons LoRa**.

## Fonctions principales

- Point d’accès Wi-Fi local avec tableau de bord web embarqué
- Configuration du nœud depuis un navigateur
- Émission manuelle ou périodique de balises LoRa
- Réception et suivi des nœuds voisins
- Mesures radio :
  - RSSI
  - SNR
  - erreur de fréquence
  - paquets manqués
  - doublons
- Journalisation CSV par session dans LittleFS
- Support optionnel d’un écran OLED
- Compatible avec la **Heltec WiFi LoRa 32 V2** et d’autres cartes **ESP32 + LoRa**

## Cas d’usage typique

Ce firmware est utile lorsque plusieurs nœuds LoRa portables sont déployés sur le terrain pour répondre à des questions comme :

- Est-ce que ces deux positions se voient correctement en LoRa ?
- Quel emplacement donne la liaison la plus stable ?
- Comment la qualité du lien varie-t-elle selon la géométrie du terrain ?
- Quel spreading factor est nécessaire pour ce trajet ?

Un smartphone peut se connecter directement au point d’accès Wi-Fi d’un nœud et afficher le tableau de bord web pour une inspection en direct.

## Interface web

Après le démarrage, chaque nœud crée un point d’accès Wi-Fi du type :

`LoRaTest-XXXX`

Ouvrir dans un navigateur :

`http://192.168.4.1`

L’interface fournit :

- **Bandeau du nœud** avec l’identité du nœud courant
- **Tableau des nœuds détectés**
- **Détails radio**
- **Panneau de configuration rapide**
- **Gestion des sessions et export des logs**
- **Rafraîchissement automatique du tableau de bord** lorsque des paquets valides sont reçus

## Règles d’exploitation radio

Tous les nœuds d’une même session doivent partager **exactement le même profil radio** :

- fréquence
- spreading factor
- bande passante
- coding rate
- puissance TX
- longueur de préambule
- sync word
- CRC

### Règle importante anti-collision

En **SF12**, prévoir au moins **10 secondes d’intervalle de transmission par nœud**.

Intervalle minimal recommandé par nœud :

| Nombre de nœuds | Intervalle minimal recommandé en SF12 |
|---|---:|
| 1 | 10 s |
| 2 | 20 s |
| 3 | 30 s |
| 4 | 40 s |

**Exemple :**  
Avec **3 nœuds**, configurer environ **30 secondes entre deux transmissions sur chaque nœud**.

Avec des spreading factors plus faibles comme **SF7** ou **SF9**, des intervalles plus courts peuvent être utilisés.

## Matériel pris en charge

### Pris en charge directement
- **Heltec WiFi LoRa 32 V2**

### Pris en charge avec adaptation du brochage
- autres cartes **ESP32 + LoRa**
- cartes **avec ou sans écran**

Pour porter le firmware sur une autre carte, il faut adapter :

- les broches SPI
- le chip select LoRa
- le reset LoRa
- la broche DIO0 / interruption
- les broches I2C si un OLED est utilisé

## Installation

### 1. Environnement de développement

Installer l’un des environnements suivants :

- **Arduino IDE 2.x**
- **PlatformIO**

Puis installer le **support de cartes ESP32 par Espressif**.

### 2. Bibliothèques

#### À installer manuellement

- **LoRa** par **Sandeep Mistry**
- **U8g2** seulement si le support OLED est activé

#### Déjà fournies par le core ESP32

Aucune installation séparée n’est normalement nécessaire pour :

- `WiFi.h`
- `WebServer.h`
- `DNSServer.h`
- `Preferences.h`
- `LittleFS.h`
- `SPI.h`
- `Wire.h`

### 3. Téléversement du firmware

#### Procédure avec l’IDE Arduino

1. Ouvrir le sketch
2. Sélectionner la carte  
   Pour la **Heltec WiFi LoRa 32 V2**, `ESP32 Dev Module` fonctionne généralement bien si les broches sont correctement définies dans le code
3. Sélectionner le bon port série
4. Compiler
5. Téléverser

Utiliser un **câble USB de données**, pas un câble de charge uniquement.

### 4. Envoi des logos / favicon / fichiers statiques

Si vous voulez afficher correctement les logos et l’icône, envoyez le dossier `data/` avec **LittleFS Filesystem Uploader** :

<https://github.com/earlephilhower/arduino-littlefs-upload>

Structure typique du projet Arduino :

```text
project/
  your_sketch.ino
  data/
    logo_haamani.png
    logo_labo.png
    logo_univ.png
    logo_terraforma.png
```

## Firmware précompilé

Un paquet de firmware précompilé pour **Heltec WiFi LoRa 32 V2** est également disponible dans les **Releases** du projet sous forme d’archive `.zip`.

Cette archive contient :

- `sketch_mar26a_LoRa_Tester_V1.ino.bootloader.bin`
- `sketch_mar26a_LoRa_Tester_V1.ino.partitions.bin`
- `sketch_mar26a_LoRa_Tester_V1.ino.bin`

Cela permet de flasher le firmware **sans compiler le projet à partir des sources**.

### Flash des binaires précompilés

La méthode la plus simple consiste à utiliser **esptool**.

Exemple de commande sous Windows :

```bash
python -m esptool --chip esp32 --port COMX --baud 460800 write_flash -z ^
  0x1000 sketch_mar26a_LoRa_Tester_V1.ino.bootloader.bin ^
  0x8000 sketch_mar26a_LoRa_Tester_V1.ino.partitions.bin ^
  0x10000 sketch_mar26a_LoRa_Tester_V1.ino.bin
```

Remplacer `COMX` par le bon port série, par exemple `COM5`.

### Étapes de flash

1. Connecter la Heltec WiFi LoRa 32 V2 avec un **câble USB de données**
2. Identifier le port série dans l’IDE Arduino ou dans le gestionnaire de périphériques
3. Extraire l’archive `.zip`
4. Ouvrir un terminal dans le dossier extrait
5. Exécuter la commande `esptool` ci-dessus
6. Attendre la fin du flash
7. Redémarrer la carte si nécessaire

### Remarques

- Toujours connecter l’**antenne LoRa** avant les essais radio
- En cas d’échec du flash, essayer un autre câble USB, vérifier le port COM, ou utiliser les boutons **PRG / RESET** pour entrer en mode bootloader
- Si le projet utilise également des fichiers web stockés dans LittleFS, ceux-ci doivent être envoyés séparément si aucune image LittleFS précompilée n’est fournie

## Support OLED

Le projet peut être compilé :

- **avec OLED** si `USE_OLED = 1`
- **sans OLED** si `USE_OLED = 0`

Quand l’OLED est désactivé :

- l’interface web continue de fonctionner
- la journalisation CSV continue de fonctionner
- la sortie série reste disponible
- la consommation électrique peut être réduite

## Démarrage rapide

1. Flasher le même firmware sur tous les nœuds
2. Donner à chaque nœud un **nom unique**
3. Régler les **mêmes paramètres radio** sur tous les nœuds
4. Choisir un **nom de session commun**
5. Régler l’intervalle d’émission selon le nombre de nœuds
6. Déployer les nœuds sur le terrain
7. Se connecter au Wi-Fi d’un nœud et consulter le tableau de bord
8. Exporter les logs CSV en fin de session

## Interprétation des indicateurs

| Indicateur | Lecture rapide | Signification |
|---|---|---|
| RSSI | Plus élevé = mieux | Puissance du signal reçu |
| SNR | Plus élevé = mieux | Rapport signal sur bruit |
| Missed | Plus faible = mieux | Estimation des paquets manqués |
| Dup | Plus faible = mieux | Doublons reçus |
| Frequency error | Proche de zéro = mieux | Peut indiquer un décalage RF ou une dérive |

## Bonnes pratiques terrain

- attribuer un **identifiant unique** à chaque nœud avant le départ
- utiliser un **nom de session explicite**
- tester tous les nœuds ensemble à courte distance avant la mission
- garder des paquets **courts** lorsque l’objectif est de mesurer la qualité du lien
- exporter les fichiers CSV après chaque session
- prévoir une **power bank par nœud**
- toujours connecter l’**antenne LoRa** avant les essais radio

## Dépannage

### Aucun nœud détecté
Vérifier :

- fréquence
- SF
- BW
- CR
- sync word
- CRC
- présence de l’antenne
- intervalle de transmission

### Très peu de paquets reçus
Cause probable :

- collisions dues à des transmissions trop rapprochées

Correction :

- augmenter l’intervalle de transmission
- respecter la règle des **10 secondes par nœud en SF12**

### Échec du téléversement
Vérifier :

- le port série
- le câble USB
- le mode bootloader
- la séquence PRG / RESET si nécessaire

### Les logos ou le favicon n’apparaissent pas
Vérifier que les fichiers ont bien été envoyés dans **LittleFS**.

### L’OLED reste vide
Vérifier :

- les broches de l’écran
- le modèle d’écran
- le réglage d’activation/désactivation de l’OLED
- le démarrage série d’abord, la radio d’abord

## Structure du projet

```text
.
├── your_sketch.ino
├── data/
│   ├── logo_haamani.png
│   ├── logo_labo.png
│   ├── logo_univ.png
│   └── logo_terraforma.png
├── README.md
├── README.fr.md
├── README.en.md
├── LICENSE
└── NOTICE
```

## Projets liés et écosystème

Ce projet s’inscrit dans un écosystème existant de cartes **ESP32 + LoRa**, mais il combine plusieurs fonctions dans une approche spécifiquement orientée **tests de ligne de vue LoRa multi-nœuds sur le terrain**.

Quelques bibliothèques et projets proches ou utiles :

- **LoRa de Sandeep Mistry**  
  Une bibliothèque Arduino LoRa très utilisée pour les radios SX127x.  
  <https://github.com/sandeepmistry/arduino-lora>

- **RadioLib**  
  Une bibliothèque plus large prenant en charge de nombreux transceivers radio, dont LoRa.  
  <https://github.com/jgromes/RadioLib>

- **Support Heltec ESP32 LoRa / projets spécifiques aux cartes**  
  <https://heltec.org>  
  <https://github.com/ropg/heltec_esp32_lora_v3>

- **LoRaMessenger**  
  Un projet ESP32 + LoRa combinant communication LoRa et interface Wi-Fi locale.  
  <https://github.com/TheNico14/LoRaMessenger>

- **LowCostLoRaGw / exemples de test de portée**  
  Un ensemble d’exemples et d’outils autour de LoRa, incluant des sketches utiles pour les tests radio et les diagnostics terrain.  
  <https://github.com/CongducPham/LowCostLoRaGw>

### Positionnement de ce projet

Par rapport aux bibliothèques LoRa génériques ou aux exemples orientés messagerie, ce firmware est surtout conçu pour :

- la **préparation de déploiements terrain**
- les **tests de visibilité radio LoRa multi-nœuds**
- l’**observation de la qualité radio via une interface web locale**
- la **journalisation CSV par session**
- une **utilisation simple sur le terrain par des non-développeurs**

Ce projet doit donc être compris comme un **outil de test terrain orienté recherche**, construit à partir de briques logicielles LoRa existantes, et non comme une simple bibliothèque générique de communication LoRa.

## Limites connues

- LoRa est **half-duplex**
- les paquets longs et les spreading factors élevés augmentent le risque de collision
- en **SF12**, les intervalles automatiques doivent être beaucoup plus longs
- le rafraîchissement web en direct est pratique, mais les vraies notifications push navigateur ne sont pas l’approche privilégiée dans cette architecture en point d’accès local
- le portage vers une nouvelle carte nécessite une vérification attentive du brochage

## Idées d’évolution

- mode télémétrie compact pour usage longue portée en SF12
- préréglages pour plusieurs bandes régionales
- mises à jour en direct via WebSocket en option
- export ZIP d’une session complète
- affichage de la tension batterie dans l’interface
- profils prédéfinis pour plusieurs cartes ESP32 LoRa courantes

## Demande de citation

Si vous réutilisez, redistribuez, modifiez ou intégrez ce logiciel ou sa documentation, merci de citer :

> Developed by Jean Martial Mari, GEPASUD, UPF.  
> Copyright TERRA FORMA.  
> Hāmani Fenua and TERRA FORMA projects.

Contexte du projet :
- Hāmani Fenua
- TERRA FORMA

## Licence

**Haamani Fenua / TERRA FORMA Attribution License 1.0**

Copyright (c) 2026 **TERRA FORMA**

**Auteurs**  
Jean Martial Mari, GEPASUD, UPF

Ce logiciel est fourni **“AS IS”**, sans garantie d’aucune sorte, expresse ou implicite.

Voir le texte complet dans le fichier `LICENSE`.

## Remerciements

- Hāmani Fenua
- TERRA FORMA
- GEPASUD
- UPF
- laboratoires partenaires et équipes de terrain
