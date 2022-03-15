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

Le protocole utilisé pour communiquer entre les microcontrolleurs STM32 et le "cerveau" du robot, la Raspberry Pi est le CAN Bus. Très utilisé en industrie notament dans le domaine de l'automobile, nous l'avons choisi car il permet de transmettre les messages avec juste 2 cables. Ce protocole n'a pas le concept de ID pour les différents machines sur le réseau mais peut être implémenté ultérieurement. En revanche, nous avons le concept de priorité de messages pour éviter les collison dans le réseau. Ce sera le messages de plus haute priorité qui sera émis en cas de conflit. Chaque trame de CAN se compose d'une zone de data de taille maximale de 64 octects. Pour avoir plus d'informations techniques sur le CAN vous pouvez [clickez ici](https://www.ti.com/lit/an/sloa101b/sloa101b.pdf?ts=1633140726383&ref_url=https%253A%252F%252Fwww.google.com%252F/)

