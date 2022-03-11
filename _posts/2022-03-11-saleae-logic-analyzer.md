---
title: Saleae logic analyzer
categories: ["2022", logic-analyzer]
tags: [post,tutorial,elec-soft,can-bus,débogage]
author:
  name: Joel IMBERGAMO GUASCH
  link: https://joelimgu.github.io/
image:
  src: https://bdspeedytech.com/image/cache/catalog/Usb%20Saleae%20Logic%20Analyzer%2024M%20Sampling%208%20Channels-750x750.png
  width: 500   # in pixels
  height: 200   # in pixels
  alt: tuto tirage cartes
pin: false
render_with_liquid: true
comments: false
math: false
mermaid: false
---

# Saleae logic analyzer

Débogage du bus CAN et protocoles de communication en général.


- Installer le software saleae (sous linux)

  ⇒ pas besoin de software additionnel pour exécuter (appimage) mais ne pas oublier d’ajouter les droits d’exécution

  [Download sur son site](https://www.saleae.com/downloads/)

- exécuter le message *cat ...* qui s’affiche à l’ouverture du programme
- fermer l’application
- la réouvrir, débrancher et rebrancher le logic analyzer du PC
- la connexion est normalement établie

Utilisation:

- Rajouter un analyzer (barre d’icônes à droite) en CAN
- ⚠ Le Pin 1 du LogicAnalizer correspond au pin 0 sur le programme!!!! ( oui c’était trop facile sinon )
- Lancer l’acquisition (Start en haut à droite)
