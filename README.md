# horloge-sebsab-firmware

Dépôt public minimal, destiné uniquement à distribuer le firmware compilé de
l'[Horloge SebSab](https://github.com/Seb-Sab/horloge-sebsab) (dépôt privé,
qui contient le code source).

Chaque horloge vérifie ce dépôt au démarrage et chaque jour à 4h31 :
- `version.txt` : un simple entier, la dernière version publiée.
- `firmware.bin` : le binaire compilé correspondant.

Si `version.txt` > version installée sur l'horloge, elle télécharge et
flashe `firmware.bin` automatiquement (LEDs 1 à 121 en dégradé rouge→vert
pendant le téléchargement), puis redémarre.

## Canaux stable/beta (depuis v68)

Deux sous-dossiers, chacun avec son propre `version.txt`/`firmware.bin` :
`stable/` et `beta/`. Chaque horloge choisit lequel suivre (`channel`,
piloté à distance depuis le dashboard fleet — voir
`horloge-sebsab-fleet/api/setConfig.js` — et stocké en EEPROM,
`EEPROM_CHANNEL_ADDR` dans `src/main.cpp`) ; par défaut toutes les
horloges sont sur `stable`.

**Les fichiers à la racine (`firmware.bin`/`version.txt`) sont conservés
en plus**, en miroir de `stable/` — nécessaire pour les horloges
pré-v68, qui ne connaissent pas encore les sous-dossiers de canal et
continueront à interroger la racine indéfiniment tant qu'elles n'ont pas
elles-mêmes reçu v68. Toute publication en canal **stable** doit donc
mettre à jour les **trois** emplacements (racine + `stable/`) ; une
publication **beta uniquement** ne touche que `beta/`.

## Publier une nouvelle version

1. Dans le dépôt source `horloge-sebsab`, monter `#define FW_VERSION` dans
   `src/main.cpp` (ex: `1` → `2`).
2. Compiler (`PlatformIO: Build` dans VSCode, ou `pio run`). Le binaire est
   généré dans `.pio/build/d1/firmware.bin`.
3. Copier ce fichier ici :
   - Version **beta** (à tester sur les horloges basculées en beta avant
     diffusion générale) : écraser seulement `beta/firmware.bin`, mettre à
     jour `beta/version.txt`.
   - Version **stable** (promue après validation, ou changement qui ne
     nécessite pas de phase beta) : écraser `firmware.bin` **et**
     `stable/firmware.bin` (même binaire aux deux emplacements), mettre à
     jour `version.txt` **et** `stable/version.txt` avec le même numéro.
4. Ajouter une ligne dans `CHANGELOG.md` (canal, date, courte description)
   — c'est ce qui permet de savoir plus tard si un numéro de version donné
   est passé par une phase beta, l'entier seul ne le dit pas. Une
   promotion beta → stable sans changement de code ajoute une seconde
   ligne pour le même numéro.
5. Commit + push de **ce dossier uniquement** vers le dépôt public. Le code
   source ne quitte jamais le dépôt privé.

Les horloges sur le canal concerné détecteront la mise à jour au prochain
redémarrage, ou au plus tard le lendemain à 4h31.

## Règle importante — ne pas casser les réglages existants

Les réglages de chaque horloge (Wi-Fi, couleur des LEDs, date d'anniversaire,
langue, fuseau horaire) sont stockés en EEPROM et survivent nativement à une
mise à jour OTA (l'EEPROM vit dans un secteur de flash séparé des
partitions de firmware). Pour que ça reste vrai :

- Ne jamais changer `EEPROM_SIZE` ni déplacer/réutiliser une adresse déjà
  occupée dans `src/main.cpp` sans écrire un code de migration (lire
  l'ancien format, le convertir, puis réécrire).
- Ne pas changer la configuration de board dans `platformio.ini`
  (`board = d1`), qui détermine l'emplacement du secteur EEPROM.

Avant de publier une version qui touche à l'EEPROM, tester sur une horloge
réelle : configurer tous les réglages, déclencher l'OTA, vérifier qu'ils
sont toujours corrects après redémarrage.
