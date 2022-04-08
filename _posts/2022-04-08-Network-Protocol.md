---
title: Network Protocol Definition
categories: ["2022", can-bus]
tags: [elec-soft,can-bus,info, network]
author:
  name: Joel Imbergamo Guasch
  link: https://github.com/joelimgu
pin: false
render_with_liquid: false
comments: false
math: false
mermaid: false
---


Ce document décrit le fonctionnement du protocole utilisé pour les communications entre les microcontroleurs du robot et la raspi.

C'est un protocole sans pertes c.-à-d. toutes les trames sont vérifiées avec un paquet d'acquittement avant de les marqués comme bien envoyés.


| Trame sur 64 bits |                                       |         |         |     |      |
|-------------------|---------------------------------------|---------|---------|-----|------|
| 4b                | 4b                                    | 3b      | 4b      | 1b  | 48b  |
| ID_dest           | ID_sender                             | ID_mess | seq_num | ACK | data |


| Name      | Type            | Explication                                                                                                                                                                                                                                                                 |
|-----------|-----------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| ID_dest   | unsigned (4bit) | ID de la STM auquelle le message va dirigé                                                                                                                                                                                                                                  |
| ID_sender | unsigned (4bit) | ID de la STM qui envoie le message                                                                                                                                                                                                                                          |
| ID_mess   | unsigned (3bit) | Identificateur unique du message. La combinaison (ID_sender,ID_des,ID_mess) doit être unique. cela veut dire qu'on ne peux pas envoyer un message avec la même idée tant qu'on a pas liberée la ID ( la ID est liberée quand on a reçu tous les ACK des trames du message ) |
| seq_num   | unsigned (4bit) | Numero de sequence de la trame. ⚠ On le decremente. Ce numero indique la l'ordre de la trame pour permetre de reconstruire le message bien ordoné. On le decremante, cela implique que la première trame indique le número de trames qui forment le message.                |
| ACK       | bool (1b)       | Indique si la trame est un paquet d'acquittement                                                                                                                                                                                                                            |


# Examples

STM(ID:4) <-------------------> STM(ID:12)

STM4 evnoye un message à STM12:

Notre message: Club_Robot_INSAT

Taille du message = 16bytes ( codé en ASCII )

Cela veut dire qu'il faut diviser notre message en 3 trames de 6 bytes. ( 16 / 6 = 3 ) ( on pourrait coder des espaces sans problemme mais c'est compliquer à les écrire d'una façon claire pour l'example 😅)

3 messages => seq_num = 2 de la première trame

On prend l'id_mess = 0, car c'est le premier message qu'on evoye.

Trames envoyés par le STM4:
1. [12,4,0,2,0,C,l,u,b,_,R]
2. [12,4,0,1,0,o,b,o,t,_,I]
3. [12,4,0,0,0,N,S,A,T,0,0] -> On rempli avec des 0's le reste du message

Trames envoyées par le STM12 ( ACK ):
1. [4,12,0,2,1,0,0,0,0,0,0]
2. [4,12,0,1,1,0,0,0,0,0,0]
3. [4,12,0,0,1,0,0,0,0,0,0]

Si maintenant on veut envoyer un autre message il faut juste augmenter l'id_mess mod 7.






