---
title: "Notre microcontrôleur: le STM32F103"
categories: ["2022", can-bus]
tags: [post,tutorial,elec-soft, stm32, microcontroleur]
author:
  name: Andrea PEREZ FERNANDEZ
  link: https://github.com/andreapefe
pin: false
render_with_liquid: false
comments: false
math: false
mermaid: true
---

Pour gérer les actionneurs, moteurs et capteurs, nous utilisons des microcontrôleurs répartis dans tout le robot. Parmi ceux existant nous utilisons des STM32F103 comme dans les TPs de 4A à l'INSA.

Un microcontrôleur est un ”petit” maid qui ne possède pas de système d’exploitation, il est vide. Ainsi, c'est nous qui devons écrire le code qui va s’exécuter sur lui, le firmaware (entre soft et hardware) puis le transmettre à ce dernier.
Cette action se nomme flasher le contrôleur. Une fois le contrôleur a été flashé il garde le programme à exécuter dans sa mémoire morte et une fois mis sur tension il s'exécutera.

Si vous observez les microcontrôleurs au Club, vous vous rendrez compte qu’ils possèdent un gran nombre de pins (48 dans notre cas). Chacun porte un nom spécial et a aussi des fonctions désignées. Nous avons des pins de GND, de source de tension, mais aussi des GPIOs (General Purpose Input Output) qui nous permettent de communiquer avec l’extérieur. Ces derniers peuvent être configurés par le programmeur pour répondre à ses besoins. Pour fixer cette configuration, nous devons affecter certaines valeurs spéciales aux registres de control de chacun de
ses ports. Grâce à la librairie HAL (High Applicative Layer) nous n'aurons pas besoin de le faire directement, mais en appelant leurs méthodes. Regarder le code firmware commenté pour comprendre les étapes de configuration.

De plus, selon le périphérique que nous souhaitons utilisé ces ports vont devoir être configuré selon une configuration précise. Pour trouver toutes les configurations pour chaque périphérique il faut regarder dans le reference manual [RM0008](https://www.google.com/url?sa=t&rct=j&q=&esrc=s&source=web&cd=&cad=rja&uact=8&ved=2ahUKEwjO74XN7cj2AhWeQkEAHeoIDS8QFnoECAUQAQ&url=https%3A%2F%2Fwww.st.com%2Fresource%2Fen%2Freference_manual%2Fcd00171190-stm32f101xx-stm32f102xx-stm32f103xx-stm32f105xx-and-stm32f107xx-advanced-arm-based-32-bit-mcus-stmicroelectronics.pdf&usg=AOvVaw2kF0T1D3TzsgvgnX7fvMku) à la page *166*. Pour voir sur quels pins sont branchés tous les périphériques (USART, Timer, etc..) il faut voir la page *176*.

Les différentes configurations des ports GPIO possibles sont :

| Mode Input         | Utilisation                                                                                                                |
|--------------------|----------------------------------------------------------------------------------------------------------------------------|
| Floating Input     | Mode standard, prend l’entrée tel quel sans tensions de  référence                                                         |
|Input with pull-up/pull-down | Entrée digital avec réference au GND ou à la tension de polarisation (Vcc) selon à quel potentiel on branche la résistance |
| Analog Input       | Pour signaux analogiques (ADC ou CAN)                                                                                      |


| Mode Output        | Utilisation                                  |
| -------------------|----------------------------------------------|
| General purpose output push-pull | Le composant sera alimenté par pin de sortie |
| General purpose output Open-drain| Composant alimenté par nous. Besoin d’une résistance pull-up (au VCC) pour assurer le niveau logique ’1’ |
| Alternate function output Push-pull | Configuration spéciale pour utiliser certains périphériques |
| Alternate function output Open-drain | Configuration spéciale pour utiliser certains périphériques. Vérifier le besoin d'un résistance pull-up |


Dernier point important sur le clock. C'est le signal qui synchronise et fait marcher tout le système. Ainsi vous observerez dans le code qu'avant d'utiliser n'importe quel GPIO il faut alimenter leur branche par une clock pour les activer. Chaque périphérique est sur une des deux branches qui existent avec chacune une fréquence maximale différente de fonctionnement (ceci est important pour les protocoles de communication notamment de type USART).




