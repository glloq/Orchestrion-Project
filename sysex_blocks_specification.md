# MidiMind SysEx Protocol - Spécification Réponses

## VUE D'ENSEMBLE

### Protocole Custom MidiMind
- **ID Manufacturer** : 0x7D (Educational/Development)
- **Sous-ID** : 0x00 (MidiMind)
- **Format général** : `F0 7D 00 <BlockID> <Type> <Data> F7`

### Séquence d'Identification

```
1. Standard Identity Request  → F0 7E 7F 06 01 F7
   Réponse optionnelle        → Standard MIDI Identity Reply

2. Block 1 Request            → F0 7D 00 01 00 F7
   Réponse obligatoire        → Identification instrument

3. Block 2 Request            → F0 7D 00 02 00 F7
   Réponse optionnelle        → Capacités avancées
```

---

## BLOCK 1 : IDENTIFICATION INSTRUMENT

### Objectif
Identifier l'instrument et déclarer quelles notes MIDI il peut jouer.

### Structure Complète

```
F0 7D 00 01 01 <Version> <Name[16]> <GM> <FirstNote> <Count> <Poly> <Flags> [NoteBitmap] F7

Offset  Taille  Champ           Description
------  ------  -----           -----------
0-4     5       Header          F0 7D 00 01 01
5       1       Version         0x01 (format version)
6-21    16      Name            Nom ASCII, null-padded
22      1       GM Program      0-127 (0xFF si non-GM)
23      1       First Note      0-127 (première note)
24      1       Note Count      1-127 (nombre de notes)
25      1       Polyphony       1-127 (max notes simultanées)
26      1       Flags           Bitfield (voir ci-dessous)
27-45   0/19    Note Bitmap     Conditionnel (si flags bit 0 = 0)
46/27   1       End             F7
```

### Flags Byte (Offset 26)

| Bit | Nom | Description |
|-----|-----|-------------|
| 0 | CONSECUTIVE_NOTES | 1=Notes consécutives (pas de bitmap), 0=Bitmap suit |
| 1-7 | Réservés | Toujours 0 |

### Logique Notes

**Si CONSECUTIVE_NOTES = 1** :
- Notes jouables : `FirstNote` à `FirstNote + Count - 1`
- Pas de bitmap envoyé
- **Taille message : 28 bytes**

**Si CONSECUTIVE_NOTES = 0** :
- Notes arbitraires définies par bitmap 128 bits
- Bitmap de 19 bytes suit (encodage 7-bit)
- **Taille message : 47 bytes**

### Note Bitmap (Encodage 7-bit)

**Format** : 128 bits (16 bytes) → encodé en 19 bytes pour SysEx

```
Bitmap natif (16 bytes):
  Byte 0 = Notes 0-7 (bit 0 = note 0, bit 7 = note 7)
  Byte 1 = Notes 8-15
  ...
  Byte 15 = Notes 120-127

Encodage SysEx (19 bytes):
  Bytes 0-15  : Bits 0-6 de chaque byte source
  Bytes 16-18 : Bits 7 (MSB) packés
```

**Exemple bitmap** : Notes 36, 38, 42
```
Byte 4 (notes 32-39): 
  Bit 4 (note 36) = 1
  Bit 6 (note 38) = 1
  → Byte 4 = 0x50

Byte 5 (notes 40-47):
  Bit 2 (note 42) = 1
  → Byte 5 = 0x04

Après encodage 7-bit → 19 bytes SysEx
```

---

## BLOCK 2 : CAPACITÉS

### Objectif
Déclarer les capacités avancées de l'instrument (CC, aftertouch, pitch bend, etc.).

### Structure Complète

```
F0 7D 00 02 01 <Version> <CapFlags[2]> [CCBitmap] F7

Offset  Taille  Champ           Description
------  ------  -----           -----------
0-4     5       Header          F0 7D 00 02 01
5       1       Version         0x01 (format version)
6-7     2       Cap Flags       16 bits capacités (encodé 7-bit)
8-26    0/19    CC Bitmap       Conditionnel (si bit 0 de flags = 1)
27/8    1       End             F7
```

### Capability Flags (16 bits)

| Bit | Nom | Description |
|-----|-----|-------------|
| 0 | CC_SUPPORT | 1=Supporte CC (bitmap suit), 0=Pas de CC |
| 1 | VELOCITY_CURVES | 1=Courbes vélocité custom |
| 2 | CHANNEL_AFTERTOUCH | 1=Aftertouch canal |
| 3 | POLY_AFTERTOUCH | 1=Aftertouch polyphonique |
| 4 | PROGRAM_CHANGE | 1=Program change |
| 5 | PITCH_BEND | 1=Pitch bend |
| 6 | SYSEX_CONFIG | 1=Config via SysEx |
| 7 | MPE_SUPPORT | 1=MIDI Polyphonic Expression |
| 8 | NRPN_SUPPORT | 1=NRPN messages |
| 9 | RPN_SUPPORT | 1=RPN messages |
| 10-15 | Réservés | Toujours 0 |

**Encodage SysEx** :
```
Flags 16-bit: 0xABCD
SysEx Byte 0: 0xABCD & 0x7F = bits 0-6
SysEx Byte 1: (0xABCD >> 7) & 0x7F = bits 7-13
```

### Logique CC

**Si CC_SUPPORT = 1** :
- Bitmap CC de 19 bytes suit
- Format identique au bitmap notes
- **Taille message : 28 bytes**

**Si CC_SUPPORT = 0** :
- Pas de CC supporté
- Pas de bitmap
- **Taille message : 9 bytes**

### CC Bitmap (même encodage que notes)

**Exemple** : CC 1, 7, 10, 11 supportés
```
Byte 0 (CC 0-7):
  Bit 1 (CC 1) = 1
  Bit 7 (CC 7) = 1
  → Byte 0 = 0x82

Byte 1 (CC 8-15):
  Bit 2 (CC 10) = 1
  Bit 3 (CC 11) = 1
  → Byte 1 = 0x0C

Bytes 2-15 = 0x00

Après encodage 7-bit → 19 bytes SysEx
```

---

## EXEMPLES COMPLETS

### Exemple 1 : Drum Kit Simple

**Configuration** :
- Nom : "DrumKit Pro"
- Notes : 36-51 (16 notes consécutives)
- Polyphonie : 16
- CC : 1, 7, 10, 11

**Block 1 Reply (28 bytes)** :
```
F0 7D 00 01 01               Header + Reply
01                           Version 1
44 72 75 6D 4B 69 74 20      "DrumKit Pro" (11 chars)
50 72 6F 00 00 00 00 00      + padding (5 nulls)
FF                           GM: N/A (0xFF)
24                           First note: 36
10                           Count: 16
10                           Polyphony: 16
01                           Flags: CONSECUTIVE
[pas de bitmap]
F7                           End

Total: 28 bytes
```

**Block 2 Reply (28 bytes)** :
```
F0 7D 00 02 01               Header + Reply
01                           Version 1
01                           Flags byte 0: CC_SUPPORT (bit 0 = 1)
00                           Flags byte 1: rien
[19 bytes CC bitmap]         CC 1,7,10,11
F7                           End

Total: 28 bytes
```

**Total identification : 56 bytes**

---

### Exemple 2 : Synthé Minimaliste

**Configuration** :
- Nom : "Mini Synth"
- Notes : 0-127 (toutes, consécutives)
- Polyphonie : 8
- Capacités : Pitch bend uniquement (pas de CC)

**Block 1 Reply (28 bytes)** :
```
F0 7D 00 01 01               Header
01                           Version
4D 69 6E 69 20 53 79 6E      "Mini Synth" (10 chars)
74 68 00 00 00 00 00 00      + padding (6 nulls)
51                           GM: 81 (Lead Sawtooth)
00                           First note: 0
7F                           Count: 127
08                           Polyphony: 8
01                           Flags: CONSECUTIVE
[pas de bitmap]
F7

Total: 28 bytes
```

**Block 2 Reply (9 bytes)** :
```
F0 7D 00 02 01               Header
01                           Version
20                           Flags byte 0: bit 5 = PITCH_BEND
00                           Flags byte 1: rien
[pas de bitmap CC]
F7

Total: 9 bytes
```

**Total identification : 37 bytes**

---

### Exemple 3 : Contrôleur MIDI

**Configuration** :
- Nom : "MidiControl 32"
- Notes : Aucune (contrôleur pur)
- CC : 0-31 (32 CC consécutifs)

**Block 1 Reply (28 bytes)** :
```
F0 7D 00 01 01               Header
01                           Version
4D 69 64 69 43 6F 6E 74      "MidiControl 32" (14 chars)
72 6F 6C 20 33 32 00 00      + padding (2 nulls)
FF                           GM: N/A
00                           First note: 0
00                           Count: 0 (pas de notes)
00                           Polyphony: 0
01                           Flags: CONSECUTIVE (ignoré car count=0)
F7

Total: 28 bytes
```

**Block 2 Reply (28 bytes)** :
```
F0 7D 00 02 01               Header
01                           Version
01                           Flags: CC_SUPPORT
00                           Flags byte 1
[19 bytes CC bitmap]         CC 0-31 (premiers 4 bytes = 0xFF)
F7

Total: 28 bytes
```

**Total identification : 56 bytes**

---

### Exemple 4 : Instrument Notes Arbitraires

**Configuration** :
- Nom : "White Keys"
- Notes : Touches blanches uniquement (C, D, E, F, G, A, B répétées)
- Polyphonie : 8
- Pas de CC

**Block 1 Reply (47 bytes)** :
```
F0 7D 00 01 01               Header
01                           Version
57 68 69 74 65 20 4B 65      "White Keys" (10 chars)
79 73 00 00 00 00 00 00      + padding (6 nulls)
00                           GM: 0 (Acoustic Piano)
00                           First: 0 (ignoré)
00                           Count: 0 (=utiliser bitmap)
08                           Polyphony: 8
00                           Flags: BITMAP (bit 0 = 0)
[19 bytes note bitmap]       C,D,E,F,G,A,B pattern
F7

Total: 47 bytes
```

**Block 2 Reply (9 bytes)** :
```
F0 7D 00 02 01               Header
01                           Version
20                           Flags: PITCH_BEND uniquement
00                           Flags byte 1
[pas de CC]
F7

Total: 9 bytes
```

**Total identification : 56 bytes**

---

## TABLEAU RÉCAPITULATIF

### Tailles Messages

| Type Instrument | Block 1 | Block 2 | Total |
|----------------|---------|---------|-------|
| Drum (consécutif + CC) | 28 | 28 | **56 bytes** |
| Drum (bitmap + CC) | 47 | 28 | **75 bytes** |
| Synth simple (consécutif, pitch only) | 28 | 9 | **37 bytes** |
| Synth complet (bitmap + CC) | 47 | 28 | **75 bytes** |
| Controller (pas de notes + CC) | 28 | 28 | **56 bytes** |

### Champs Obligatoires vs Optionnels

**Block 1** :
- ✅ Obligatoire : Version, Name, GM, FirstNote, Count, Poly, Flags
- ⚠️ Conditionnel : NoteBitmap (si flags bit 0 = 0)

**Block 2** :
- ✅ Obligatoire : Version, CapabilityFlags
- ⚠️ Conditionnel : CCBitmap (si flags bit 0 = 1)
- 🔮 Futur : VelocityCurves, AftertouchData (si flags bits correspondants)

---

## RÈGLES IMPORTANTES

### Encodage ASCII
- **Name field** : 16 bytes, ASCII 7-bit (0x20-0x7E)
- Null-terminated ou null-padded
- Caractères recommandés : A-Z, 0-9, espace, tiret

### Valeurs Spéciales

| Champ | Valeur | Signification |
|-------|--------|---------------|
| GM Program | 0xFF | Non-GM / N/A |
| Note Count | 0 | Utiliser bitmap (pas consécutif) |
| Polyphony | 0 | Pas de notes (contrôleur pur) |

### Contrainte SysEx
- **Tous les bytes** doivent être 0x00-0x7F (bit 7 = 0)
- Bitmaps encodés en 7-bit (16 bytes → 19 bytes)
- Flags multi-bytes encodés en 7-bit

### Ordre Requêtes

```
1. MidiMind envoie Block 1 Request
2. Instrument répond Block 1 Reply (2s max)
3. MidiMind envoie Block 2 Request
4. Instrument répond Block 2 Reply (2s max) ou silence si non supporté
```

**Timeout** : 2 secondes par requête
**Retry** : Aucun (si timeout = device ne supporte pas)

---

## COMPATIBILITÉ

### Version 1.0 du Format

- **Block 1 version** : 0x01
- **Block 2 version** : 0x01
- Versions futures backward compatible via version field

### Devices Partiels

**Acceptable** :
- Répondre Block 1 uniquement (Block 2 optional)
- Block 2 minimal (flags = 0, pas de bitmap)

**Non-acceptable** :
- Ne pas répondre Block 1 si request reçue
- Envoyer données corrompues (bytes > 0x7F)
- Dépasser 2s de délai

---

## VALIDATION

### Checklist Instrument

- [ ] Block 1 répond en < 2s
- [ ] Name : 16 bytes, ASCII, null-padded
- [ ] GM Program : 0-127 ou 0xFF
- [ ] FirstNote, Count, Poly : ranges valides
- [ ] Flags : bit 7 = 0
- [ ] Si CONSECUTIVE : pas de bitmap envoyé
- [ ] Si bitmap : exactement 19 bytes
- [ ] Block 2 répond en < 2s (ou timeout OK)
- [ ] Capability flags : encodé 7-bit (2 bytes)
- [ ] Si CC_SUPPORT : bitmap 19 bytes
- [ ] Tous bytes message < 0x80

### Outils Test

**Python test script** :
```python
# Envoyer requests, capturer replies
# Vérifier tailles, encodage, timeouts
```

**MIDI Monitor** :
- Vérifier hex dump des messages
- Confirmer F0...F7, pas de bytes > 0x7F

---

## RÉSUMÉ 1 PAGE

```
BLOCK 1 : QUI ES-TU ?
├─ Nom instrument (16 chars)
├─ GM Program (0-127, 0xFF=N/A)
├─ Notes jouables
│  ├─ Consécutives : FirstNote + Count
│  └─ Arbitraires : Bitmap 128 bits (19 bytes)
└─ Polyphonie (1-127)

Taille : 28 bytes (consécutif) ou 47 bytes (bitmap)

BLOCK 2 : QUE PEUX-TU FAIRE ?
├─ Capability Flags (16 bits)
│  ├─ CC Support
│  ├─ Velocity Curves
│  ├─ Aftertouch
│  ├─ Pitch Bend
│  └─ MPE, NRPN, RPN...
└─ CC Bitmap (si supporté, 128 bits = 19 bytes)

Taille : 9 bytes (minimal) ou 28 bytes (avec CC)

TOTAL TYPIQUE : 37-75 bytes
TIMING : < 2s par réponse
ENCODAGE : 7-bit (0x00-0x7F)
```

---

**Version** : 1.0  
**Date** : 2025-01-22  
**MidiMind Team**
