---
title: CAN Bus 
date: 2022-03-15 19:28:00 +0800
categories: ["2022", can-bus]
tags: [post, elec-soft, can-bus, ]
author:
  name: Andrea PEREZ FERNANDEZ
  link: https://github.com/andreapefe
pin: false
render_with_liquid: false
comments: false
math: false
mermaid: true
---
# 1. Presentation du protocole
Le protocole utilisé pour communiquer entre les microcontrôleurs STM32 et le "cerveau" du robot, la Raspberry Pi est le CAN Bus. Très utilisé en industrie notamment dans le domaine de l'automobile, nous l'avons choisi car il permet de transmettre les messages avec juste 2 cables nommé CAN High et CAN Low. 

## 1.1. Avantages
Ce protocole a de nombreux avantages : 
- Il présente une grande fiabilité. En effet, il implémente 5 moyens différents de détection d'erreurs et est capable de détecter une erreur en 20 bits. Ainsi pour notre débit de 115Kb/,  une erreur est détecté moins de 1 ms. Elles sont de plus très rares grâce aux propriétés électromagnétiques du bus. 
- L'acquittement de reception est gérer par le hardware associé au CAN. DOnc il n'y a pas besoin de traiter les ACK logiciellement. Le renvoi de message en cas d'échec d'envoi se fait aussi par hardware. 
- Débits suffisamment élevés pouvant atteindre un 1 Mbit/s. 
- Gestion des collisions entre messages. Si plusieurs machines sur le réseau décide d'envoyer un message en même temps, le message le plus prioritaire sera envoyer en premier (cf. partie 1.2.). 

## 1.2. Format des trames
Les trames CAN sont composé de 2 parties, la partie ID et la partie données. Cependant, le concept de ID ne correspond pas à l'idée d'avoir un identificateur pour les différentes machines du réseau. En revanche, il se réfère à la priorité des messages qui permet de choisir quel message sera envoyer en premier en cas de collisions dans le réseau. Ce sera le message de plus haute priorité qui sera émis en cas de conflit. Dans la documentation, on se réfère à cette priorité comme l'identificateur de message (ID) même si ce n'est pas exactement ça. Il existe deux versions de ID (priorité) selon la quel la taille du champ priorité/ID des messages change :
- Standard CAN ID: Taille de 11 bits
- Extended CAN ID: Taille de 29 bits

La zone data a une taille maximale entre 0 et 8 octets soit 64 bits au total. 

Pour avoir plus d'informations techniques sur le CAN vous pouvez [clickez ici](https://www.ti.com/lit/an/sloa101b/sloa101b.pdf?ts=1633140726383&ref_url=https%253A%252F%252Fwww.google.com%252F/)


# 2. Utilisé le CAN BUS avec des STM32
## 2.1. Comment utilsier le périphérique bxCAN
Notre microcontroleur STM32 possède deux périphériques internes capables de faire du CAN Bus appelés bxCAN. Si on ne fait pas de remap sur notre STM32 ils sont accesibles via les ports suivants:

| nº CAN | TXD  | RXD  |
|--------|------|------|
| CAN 1  | PA12 | PA11 |
| CAN 2  | PB13 | PB12 |

Pour les autres ports regarder la page 176 de [la doc de la STM32F103](https://www.google.com/url?sa=t&rct=j&q=&esrc=s&source=web&cd=&cad=rja&uact=8&ved=2ahUKEwjO74XN7cj2AhWeQkEAHeoIDS8QFnoECAUQAQ&url=https%3A%2F%2Fwww.st.com%2Fresource%2Fen%2Freference_manual%2Fcd00171190-stm32f101xx-stm32f102xx-stm32f103xx-stm32f105xx-and-stm32f107xx-advanced-arm-based-32-bit-mcus-stmicroelectronics.pdf&usg=AOvVaw2kF0T1D3TzsgvgnX7fvMku)

Ils peuvent être configurés pour fonctionner avec des Standard ou Extended ID. 

Cependant, on ne peut pas l'utiliser tout seul, il faut lui associé un composant appelé CAN Transreceiver pour pouvoir s'en servir. Nous utilisons le modèle [MCP2551](https://ww1.microchip.com/downloads/en/DeviceDoc/20001667G.pdf) qui est l'ancienne version du MCP2552 utilisé mais qui marche toujours. Il est branché au CAN de la forme suivante:

![Connexions MCP2551](/assets/img/posts/CAN-bus/schema_mcp.png)

Vous observerez que nous connectons les deux poins CANH et CANL sur un circuit fermé par deux résistances de 120 Ohm. En effet, il n'a pas de sens envoyé ou recevoir tous les messages circulent sur ce milieu et sont récupérés par toutes les STM32 si elles sont en reception. On appelle ça un circuit bouchon. 

## 2.2. Comment filtrer des messages
Ces périphériques bxCAN ont des fonctionnalités supplémentaires assez intéressantes en termes de filtrage de messages. Elles sont très utiles, car les messages sont directement filtrés par le hardware ce qui allege la charge de travail du processeur. Au club, nous utilisons filtrage par ID et/ou Masque. Il existe plusieurs banques ou configurer les filtres (cf. doc officielle) donc les possibilités sont très nombreuses. Nous n'utilisons qu'une seule banque pour filtrer nos messages parce que c'est largement suffisant pour notre application. 

Le filtrage par ID est simple il suffit d'indiquer au programme que nous voulons juste recevoir des messages avec un ID particuler. Sur rust on le configure de la forme suivante :
``` rust
const u16 ID //inf à 2¹¹ si Standard ID
let mut filters = can1.modify_filters();
filters.enable_bank(0, 
    BankCOnfig::List16([ListEntry16::data_frames_with_id(StandardId::new(MY_ID).unwarp())]));
``` 
Le filtrage par Masque/ID et de filtrer de forme plus fine. Elle se compose d'un masque qui nous indique quels bits il faut regarder (bit à 1) et lesquels on peut ignorer (à 0). L'ID nous donne la valeur que devrait avoir ces bits. Ainsi par exemple si nous voulons filtrer de sorte à avoir juste des messages avec des IDs pairs nous configurons notre masque et notre ID tel que (pour des ID standards à 11 bits):
- Masque : 0x001 (dernier bit à 1)
- ID : 0x000 (dernier bit à 0)

Avec cette configuration, nous allons recevoir que des messages avec un ID qui ont le dernier bit à 0 soit des ID pairs. De forme analogue si on configure les registres tels que :

- Masque : 0x001 (dernier bit à 1)
- ID : 0x001 (dernier bit à 0)

Nous allons recevoir que des messages à ID impair. 

Pour faire cette configuration nous utilisons le code suivant :
```rust
let mut filters = can1.modify_filters();
filters.enable_bank(0, Mask32::frames_with_std_id(
    StandardId::new(0x000).unwrap(), //ID
    StandardId::new(0x001).unwrap())); //masque
```

# 3. Comment tester le CAN bus
## 3.1. Carte de prototypage
Pour tester le CAN Bus avant de l'embarqué nous avons fabriqué 3 cartes de protypage (ID de carte AAB) qui permettent d'utiliser le CAN Transreceiver avec une plaquette d'essai . Vous trouverez le circuit et le Typhoon pour la fabriquée [ici](https://github.com/ClubRobotInsat/Cartes_2022/tree/master/ID_AAB_CAN_PrototypageGrand). Les MCP2551 sont des composants montés en surface (CMS) et donc on n'a pas accès aux différents PIN directement. 

## 3.2. Mise en route

Il faut brancher la carte et le bus tel qu'indiqué dans le 2.1. Il ne faut surtout pas oublier les resistances bouchons en bord du bus de 120 Ohm au pin RS. Il faut faire gaffe aussi à bien brancher la résistance de 10 kOhm, en effet, c'est elle qui permet de fixer mode de fonctionnement du MCP (High speed, Slope-control et Standby Mode). Nous sommes en Slope-Control et on a fixé une vitesse de balayage (slew-rate) de environ 23V/us. Cette vitesse de balayage limite le débit du CAN Bus qu'on fixe logiciellement. 

**Il existe déjà un banc test déjà brancher pour tester**. Donc normalement pas besion de refaire tout ceci. 

On alimente tout le circuit en 5V pour être sûr d'avoir une tension stable pour les MCP. Il faut mettre au moins 2 MCP sur le réseau. On ne peut pas tester le CAN Bus seul car il a besoin de recevoir des ACK pour être sûrs que les messages ont été bien envoyés. 

Pour commencer à envoyer des messages et à en recevoir, il faut flasher la STM32 avec le code qui est dans la branche can-bus de ce [repo Github](https://github.com/ClubRobotInsat/robot_rust_2022). Le code est commenté et devrait être semi-compréhensible. Il permet d'afficher sur la console du debugger les messages reçus. Le code pour envoyer des messages est commenté et présent dans le loop. Si votre STM est déjà alimenté par les +5V des MCP vous pouvez débrancher le pin 3V3 du câble de flash. Il n'est pas nécessaire. 

## 3.3. Configurations possibles 
Vous pouvez soit flasher deux STM entre elles et les faire communiquer mais vous pouvez aussi connecter la raspberry Pi avec son module CAN aussi. Pour voir comment faire du CAN avec la raspberry Pi se référer à [cette page]  