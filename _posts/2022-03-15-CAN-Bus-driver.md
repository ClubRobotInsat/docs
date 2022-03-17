---
title: CAN Bus 
date: 2022-03-15 19:28:00 +0800
categories: ["2022", can-bus]
tags: [post,tutorial,elec-soft, can-bus, ]
author:
  name: Andrea PEREZ FERNANDEZ
  link: https://github.com/andreapefe
pin: true
render_with_liquid: false
comments: false
math: false
mermaid: true
---

Le protocole utilisé pour communiquer entre les microcontrolleurs STM32 et le "cerveau" du robot, la Raspberry Pi est le CAN Bus. Très utilisé en industrie notamment dans le domaine de l'automobile, nous l'avons choisi car il permet de transmettre les messages avec juste 2 cables. Ce protocole n'a pas le concept de ID pour les différentes machines sur le réseau mais peut être implémenté ultérieurement. En revanche, nous avons le concept de priorité de messages pour éviter les collisons dans le réseau. Ce sera le message de plus haute priorité qui sera émis en cas de conflit. Chaque trame de CAN se compose d'une zone de data de taille maximale de 64 octects. Pour avoir plus d'informations techniques sur le CAN vous pouvez [clickez ici](https://www.ti.com/lit/an/sloa101b/sloa101b.pdf?ts=1633140726383&ref_url=https%253A%252F%252Fwww.google.com%252F/)

Notre microcontroleur STM32 possède deux périphériques internes capables de faire du CAN Bus. Si on ne fait pas de remap sur notre STM32 ils sont accesibles via les ports suivants:

| nº CAN | TXD  | RXD  |
|--------|------|------|
| CAN 1  | PA12 | PA11 |
| CAN 2  | PB13 | PB12 |

Pour les autres ports regarder la page 176 de [la doc de la STM32F103](https://www.google.com/url?sa=t&rct=j&q=&esrc=s&source=web&cd=&cad=rja&uact=8&ved=2ahUKEwjO74XN7cj2AhWeQkEAHeoIDS8QFnoECAUQAQ&url=https%3A%2F%2Fwww.st.com%2Fresource%2Fen%2Freference_manual%2Fcd00171190-stm32f101xx-stm32f102xx-stm32f103xx-stm32f105xx-and-stm32f107xx-advanced-arm-based-32-bit-mcus-stmicroelectronics.pdf&usg=AOvVaw2kF0T1D3TzsgvgnX7fvMku)

Cependant on peut pas l'utiliser tous seul, il faut lui associé un composant appelé CAN Transreceiver pour pouvoir s'en servir. Nous utilisons le modèle [MCP2551](https://ww1.microchip.com/downloads/en/DeviceDoc/20001667G.pdf) qui est l'ancienne version du MCP2552 utilisé mais qui marche toujours. Il est branché au CAN de la forme suivante:

![Connexions MCP2551](/assets/img/posts/CAN-bus/schema_mcp.png)

Vous observerez que nous connectons les deux poins CANH et CANL sur un circuit fermé par deux résistances de 120 Ohm. En effet, il n'a pas de sens envoyé ou recevoir tous les messages circulent sur ce milieu et sont récupérés par toutes les STM32 si elles sont en reception. On appelle ça un circuit bouchon. 

Pour tester le CAN Bus avant de l'embarqué nous avons fabriquée 3 cartes de protypage qui permettent d'utiliser le CAN Transreceiver avec une plaquette d'essai (ID AAB). Vous trouverez le circuit et le Typhoon pour la fabriquée [ici](https://github.com/ClubRobotInsat/Cartes_2022/tree/master/ID_AAB_CAN_PrototypageGrand). Les MCP2551 sont des composants montés en surface (CMS) et donc on n'a pas accès au différents PIN directement. 

