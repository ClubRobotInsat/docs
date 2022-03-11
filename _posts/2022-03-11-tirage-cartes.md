---
title: Tirage cartes électroniques au FabLab
categories: ["2022", tirage cartes"]
tags: [post,tutorial,elec-hard,fablab]
author:
  name: Joel IMBERGAMO GUASCH
  link: https://joelimgu.github.io/
image:
  src: https://www.ablcircuits.co.uk/wp-content/uploads/2020/10/HDI-printed-circuit-board-scaled.jpg
  width: 500   # in pixels
  height: 200   # in pixels
  alt: tuto tirage cartes
pin: false
render_with_liquid: true
comments: false
math: false
mermaid: true
---

# Tirage cartes

### 1. Créer la carte avec kikad

### 2. l’exporter comme SVG

### 3. Imprimer en sélectionnant type d’impression sur du papier transparent

Dans les propriétés d’impression il faut sélectionner du papier transparent pour que l’imprimante mette plus d’encre et bien marquer la carte sur le papier. Bien s’assurer que toutes les lignes sont continues et que l’image n’a pas de séparation entre les pixels.

### 4. Suivre le livret d’impression de cartes au fablab

1.
Couper une carte dans la taille plus petite possible ( laisser de l’espace pour coller le film sur la résine )

2.
Retirer la protection de la résine ( truc bleu collé )

3.
Coller le film avec du scotch à la résine

4.
Poser la carte avec le film collé dans l’émetteur d’ultraviolets ( face de l'impression contre les ultraviolets )

5.
Retirer le film ( le garder pour faire plus de cartes plus tard )

6.
Mettre la résine dans le releveur ( liquide orange|marron ) pour 30 s jusqu’à 2 min jusqu’à voir notre circuit apparaître sur la carte

7.
Une fois on voit notre circuit sur la carte on la nettoie à l’eau

8.
On met la carte dans l’acide ( truc bleu ) et on active tout ( pompes, chaleur, lumière et ventilation ). On la laisse dans l’acide minimum 20min et jusqu’à voir que la carte n’est plus rose|marron ( il n’y a plus de cuivre où il ne devrait pas y en avoir ). On peut laisser la carte dans l’acide autant qu’on veut tandis que le cuivre dans la zone où on veut le garder ne commence pas à s’enlever.

9.
Rincer la carte dans l’eau

10.
Passer du papier ou coton avec de l’acétone pour enlever la résine sur le cuivre restant

11.
Normalement on a une carte fonctionnelle ( pas toujours 😂 )

Par exemple cette carte n'est pas bonne, elle a encore du cuivre par tout, tout est connecté avec tout c'est inutile :

![Example carte raté](/assets/img/posts/tirage-cartes/carte-rate.jpg)

Ça c’est la bonne couleur:

![Example carte bonne](/assets/img/posts/tirage-cartes/carte-bonne.png)
