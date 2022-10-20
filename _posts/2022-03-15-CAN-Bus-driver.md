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
Le protocole utilisé pour communiquer entre les microcontrolleurs STM32 et le "cerveau" du robot, la Raspberry Pi est le CAN Bus. Très utilisé en industrie notamment dans le domaine de l'automobile, nous l'avons choisi car il permet de transmettre les messages avec juste 2 cables. 

Ce protocole n'a pas le concept de ID pour les différentes machines sur le réseau mais peut être implémenté ultérieurement. En revanche, nous avons le concept de priorité de messages pour éviter les collisons dans le réseau. Ce sera le message de plus haute priorité qui sera émis en cas de conflit. Dans la documentation il se refère à cette priorité comme l'identificateur de message (ID) même si ce n'est pas exactement ça. Il existe deux version de pour les ID (priorité) selon laquel la taille du champ priorité/ID des messages change :
- Standard CAN ID: Taille de 11 bits
- Extended CAN ID: Taille de 29 bits

Chaque trame de CAN se compose en plus de son ID d'une zone de data de taille maximale de 64 octects. Pour avoir plus d'informations techniques sur le CAN vous pouvez [clickez ici](https://www.ti.com/lit/an/sloa101b/sloa101b.pdf?ts=1633140726383&ref_url=https%253A%252F%252Fwww.google.com%252F/)

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

Masque : 0x001 (dernier bit à 1)

ID : 0x001 (dernier bit à 0)

Nous allons recevoir que des messages à ID impair. 

Pour faire cette configuration nous utilisons le code suivant :
```rust
let mut filters = can1.modify_filters();
filters.enable_bank(0, Mask32::frames_with_std_id(
    StandardId::new(0x000).unwrap(), //ID
    StandardId::new(0x001).unwrap())); //masque
```

# 3. Comment tester le CAN bus
Pour tester le CAN Bus avant de l'embarqué nous avons fabriquée 3 cartes de protypage qui permettent d'utiliser le CAN Transreceiver avec une plaquette d'essai (ID AAB). Vous trouverez le circuit et le Typhoon pour la fabriquée [ici](https://github.com/ClubRobotInsat/Cartes_2022/tree/master/ID_AAB_CAN_PrototypageGrand). Les MCP2551 sont des composants montés en surface (CMS) et donc on n'a pas accès aux différents PIN directement. 

