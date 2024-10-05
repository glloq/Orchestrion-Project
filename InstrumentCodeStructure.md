# Structure des intruments 

Je vais essayer de synthetiser les besoins de chaque instrument et comment sera structuré les differents code pour le projet 

## Strucure generale du code 

Chacun des code du projet pour les instruments sera codé en c++ de facons orienté objet.  

L'objectif est d'avoir une structure generale du code similaire pour tout les instruments de musique du projet :
- un fichier Setting.h : regroupe tout les reglages et parametres de l'instrument
- Un objet midiHandler.h : recoit et traite les messages midi (noteOn, noteOff, CC, SysEx) puis apelle la fonction adapté a l'instrument
- Un objet Instrument.h : gere les differentes actions a faire sur l'instrument

les instrument viendrons executer les messages recu peu importe le canal => la gestion des messages sera faite via l'IHM via serial, on viendra envoyer les messages uniquement a l'instrument voulu.

  
## messages midi utilisé 

### NoteOn et noteOff
C'est le minimum pour faire fonctionner l'instrument, il faut aussi utiliser la velocité quand c'est possible !


### les Controls Changes

Ces CC sont polyvalents et permettent de gérer les aspects essentiels de la dynamique et des effets sonores, quelle que soit la catégorie de l'instrument :
- CC1 (Modulation Wheel) : Contrôle de la modulation, du vibrato ou d'autres effets dynamiques pour tous les instruments.
- CC2 (Breath Controller) : Utile pour les instruments à vent ou pour simuler des effets de souffle sur des synthés ou autres instruments.
- CC7 (Volume) : Contrôle général du volume, utile pour tous les types d'instruments.
- CC11 (Expression) : Ajuste l'intensité du son, permettant des variations fines et expressives pour presque tous les instruments.

  
On pourra ajouter quelques cas particulier pour les instrument a cordes :
- CC5 (Portamento Time) : Pour ajuster les transitions entre les notes (glissando)
- CC74 (Brightness) : Modifie la brillance ou l'attaque du son, utile pour ajuster la texture des frottements d'un archet.


### les SysEx
Nous utiliserons les messages sysEx pour permettre un indentification automatique du type d'intrument sur l'interface de controle.

=> c'est à definir proprement!! 


# code MidiHandler.h





