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

## Publier une nouvelle version

1. Dans le dépôt source `horloge-sebsab`, monter `#define FW_VERSION` dans
   `src/main.cpp` (ex: `1` → `2`).
2. Compiler (`PlatformIO: Build` dans VSCode, ou `pio run`). Le binaire est
   généré dans `.pio/build/d1/firmware.bin`.
3. Copier ce fichier ici, en écrasant `firmware.bin`.
4. Mettre à jour `version.txt` avec le même numéro que `FW_VERSION`.
5. Commit + push de **ce dossier uniquement** vers le dépôt public. Le code
   source ne quitte jamais le dépôt privé.

Toutes les horloges en service détecteront la mise à jour au prochain
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
