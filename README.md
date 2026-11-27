<p align="center">
  <img src="https://github.com/user-attachments/assets/b81d6d66-3b00-426e-8976-34b848ff685d" height="180" alt="image" />
  &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
  <img src="https://github.com/user-attachments/assets/94f8a69d-27ae-49c5-aebd-e0bfb7a4bce7" height="180" alt="image" />
  &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
  <img src="https://github.com/user-attachments/assets/ea5eb6ac-3cab-4cc1-82c8-d74329b1851a" height="180" alt="LOGO HEPL" />
</p>

# Projet réseau embarqué sismique
## Introduction
Ce dépôt contient notre projet développé tout au long du cours de programmation µC et réseau. Le but est de concevoir un système distribué sur cartes NUCLEO-F767ZI capable de détecter des secousses sismiques dans un bâtiment à l'aide d'un accéléromètre (ADXL335) et de plusieurs modules externes reliés via les bus I2C, SPI et UART.

Chaque carte gère ses tâches en parallèle avec FreeRTOS afin d'acquérir les mesures à 100 Hz, calculer les valeurs RMS, horodater et sauvegarder les maxima en FRAM avec une RTC (synchronisée via NTP au démarrage), puis valider une secousse collective avant de déclencher l'alerte. En parallèle, une architecture pair-à-pair sur Ethernet permet aux STM32 de se découvrir en UDP et d'échanger leurs données en JSON via TCP. Le projet est divisé en 3 étapes qui seront mergées dans la branche principale du dépôt. Chaque étape aura un dossier avec les codes et les explications sur notre méthodologie et notre travail.

## Matériel utilisé
Le projet intègre un microcontrôleur STM32, le cœur de notre projet, un câble Ethernet avec un switch qui permet la connexion au réseau et la partie réseau du projet, un accéléromètre à 3 axes, une mémoire FRAM et un module RTC.

### Carte NUCLEO-F767ZI
La carte NUCLEO utilisée est la carte NUCLEO-F767ZI. Elle possède une connexion Ethernet idéale pour notre projet et une horloge maximale de 216 MHz, bien plus élevée que celle du NUCLEO-F411RE utilisé auparavant (limité à 84 MHz).

<p align="center">
  <img width="474" alt="image" src="https://github.com/user-attachments/assets/f4823be7-919a-4ce7-adc2-7e194f3ea997" />
</p>

### Accéléromètre 3 axes analogique (ADXL335) 
Pour la détection sismique, on utilise un accéléromètre à 3 axes qui permet de déterminer si les vibrations correspondent à une secousse sismique. Les 3 axes sont connectés à l'ADC du STM32 afin de quantifier leur état.

<p align="center">
  <img width="520" alt="image" src="https://github.com/user-attachments/assets/64333745-243f-43bc-860e-f798dbcc9454" />
</p>

### FRAM (module FRAM 2 Click basé sur l’IC CY15B104Q en SPI) 
Pour enregistrer les données, on utilise la mémoire FRAM 2 Click de MikroElektronika. Pour communiquer avec la FRAM, on passe par le bus SPI.

<p align="center">
  <img width="260" alt="image" src="https://github.com/user-attachments/assets/b03f3f8d-f3bb-4af7-81c2-4e89f925607b" />
</p>

### RTC (RTC 3 Click basé sur l’IC BQ32000 en I2C)
Enfin, comme outil de synchronisation temporelle, on utilise le module RTC 3 Click, également de MikroElektronika, via le bus I2C. Il possède aussi une capacité de secours très utile : si l'alimentation se coupe, la RTC continue de compter.

<p align="center">
  <img width="474" alt="image" src="https://github.com/user-attachments/assets/fa851b11-d10e-47e0-9b09-d69d919df8a8" />
</p>

## Étape 1
### Introduction
La première étape correspond aux premiers pas effectués pour faire fonctionner le projet :

* **Tâche maître :** permet de démarrer ou d'arrêter les autres tâches à l'aide d'un appui sur un bouton poussoir (BP) présent sur la carte.
* **Acquisition accéléromètre :** réalise l'acquisition des trois axes via des transferts DMA, avec une fréquence d'échantillonnage fixée à 100 Hz.
* **Heartbeat :** génère un heartbeat afin de vérifier le bon fonctionnement du programme.
* **UART de debug :** génère des messages de debug provenant des différentes tâches.
* **Broadcast de présence :** envoie périodiquement des messages broadcast pour signaler la présence de la carte sur le réseau.
## Étape 2
### Introduction
La deuxième étape ajoute le traitement des mesures et la communication point-à-point entre les cartes :

* **Traitement des données :** réalise un calcul sur les données brutes pour détecter les secousses sismiques et leurs amplitudes (moyenne glissante + valeur RMS sur une fenêtre de 1 s).
* **Tâche serveur :** répond aux requêtes point-à-point envoyées par les autres cartes.
* **Tâche client :** envoie des requêtes aux autres cartes pour récupérer leurs données.

## Étape 3
### Introduction
La troisième étape ajoute la sauvegarde persistante, la synchronisation temporelle et la validation collective avec alarme :

* **Sauvegarde en FRAM :** mémorise les dix valeurs maximales RMS ainsi que la valeur RMS max par nœud sur le réseau, et utilise les données reçues des autres nœuds pour valider une secousse détectée localement.
* **RTC et NTP :** utilise la RTC externe comme référence temporelle pour la mémorisation en FRAM, avec une synchronisation au démarrage via le serveur NTP externe.
* **Détection collective et alarme :** analyse les données reçues pour détecter une éventuelle secousse sismique collective et génère l'alarme.
