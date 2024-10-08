#  Projet de création d'un orchestre automatique

Le projet Orchestrion est une initiative ambitieuse visant à créer un orchestre composé d'instruments acoustiques actionné par des "robots".  

L'objectif final de ce projet est de permetre de jouer facilement des musiques depuis un fichier midi via des instruments qui viendrons executer les messages qu'ils recoivent.

Ce dépôt GitHub regroupera les explications générales du projet ainsi que les choix techniques.


## Contrôle de l'orchestre électronique

Tous les instruments seront contrôlés électroniquement à l'aide de microcontrôleurs Arduino (ou autre microcontroleur adapté), de moteurs dc/ pas a pas, d'électroaimants et de servomoteurs.



Le contrôle de l'orchestre électronique sera effectué à partir d'un ordinateur ou d'un Raspberry Pi via une page web. L'utilisateur pourra sélectionner un fichier MIDI et assigner les différents canaux à chaque instrument. Pour la communication MIDI via USB, nous utiliserons la bibliothèque MidiUSB.h.


# Code instruments

## Code generique
Vu que je cherche a utiliser le meme type de materiel, il n'y aura que quelques codes generique a faire pour chaque type type d'actionneurs.
les codes suivants seront construit avec des objets en c++, ils dervais etre adaptable a beaucoup d'instruments ! (ca sera très probablement que pour moi XD) 

### simple action 

[Servo MIDI Music](https://github.com/glloq/servo-midi-music)
- 1 a 128 servomoteurs avec des pca9685
- 3 type de code/fonctionnement :
  - code adaptable grattage (+/- un angle alterné)
  - action avec servomoteurs (deplacement avec noteOn, retour avec noteOff)
  - impulsion (deplacement avec noteOn, retour après un certain temps) 
- Accordage diatonique ou chromatique 

[Solenoid MIDI Music ](https://github.com/glloq/Solenoid-Midi-Music)
- Code adaptable pour l'activation de 1 a 128 solenoides 
- 3 type de code/fonctionnement :
  - action avec solenoides sans velocité ( activé avec noteOn, desactivé avec noteOff )
  - action avec solenoides avec velocité par PWM ( activé avec noteOn, desactivé avec noteOff )
  - Impulsion/Percussion ( activé avec noteOn, desactivé après un certain temps )
- accordage diatonique ou chromatique 

Stepper Motor MIDI => a venir
- Code pour controler un moteur pas a pas et gerer le deplacement a certaines position definie en fonction de la note midi recue.


### instrument a corde gratté
- instrument à corde gratté de une a 6 cordes avec X frettes par corde => [general](https://github.com/glloq/OneStringGuitar)
  - [Actionné par solenoides](https://github.com/glloq/Orchestrion_Plucked_Strings_Solenoids/tree/main)
  - [Actionné par servomoteurs](https://github.com/glloq/Orchestrion_Plucked_Strings_Servomotors/tree/main)
  - Moteur pas à pas
    - Contact Continu
    - avec servomoteurs
    - avec solenoides
      
### instrument à corde frottée
- instrument à corde frottée de une a 6 cordes avec X frettes par corde  =>[general](https://github.com/glloq/OneStringCello)

### instrument a vent
- instrument a vent "basse pression" (avec un ventilateur radial ou autre pompe a air adapté) 

    
- instrument a vent "haute pression" => c'est la partie la plus compliqué pour moi (utiliser des compresseurs de frigo ?) 

### Batteries/Percussions
l'objectif est de permettre de jouer les note midi pour un systeme classique de batteries en respectant le General MIDI PERCUSSION Key Mapping :
[percussion](https://github.com/glloq/MidiPercussion)

  -------------------------------------------------------
## Exemples d'intruments
- Xylophone, glokenspiel, marimba, vibraphone = [Code adaptable de 17 a 32 notes avec gestion pwm + silencieux pour noteOff + volume/vibrato](https://github.com/glloq/Orchestrion-Xylophone)
- [lyre 16 cordes](https://github.com/glloq/16-cords-lyre-midi)
- [ukulele solenoides](https://github.com/glloq/Orchestrion_ukulele)
- [piano](https://github.com/glloq/Orchestrion_Piano)
- [Trompette](https://github.com/glloq/Orchestrion_trumpet) 
- [flute a bec](https://github.com/glloq/servo-flute)
- [Harmonica](https://github.com/glloq/harmonica_Midi)

-------------------------------------------------------






  
