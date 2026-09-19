# Changelog

Une ligne par publication : quel canal, quand, et une description courte.
Sert à savoir, pour un numéro de version donné, s'il est passé par une
phase beta avant d'être promu stable — l'entier de version seul (voir
`version.txt`) ne le dit pas.

Format : `vN — canal — date — description`. Une version promue de beta
vers stable sans changement de code garde le même numéro : elle apparaît
deux fois (une ligne `beta` puis une ligne `stable` à la date de
promotion).

Aucune version antérieure à v68 n'a de canal — le mécanisme stable/beta
n'existe pas avant (une seule diffusion possible, implicitement
"stable"). Pas d'historique rétroactif pour ces versions-là.

## v88 — beta uniquement — 2026-09-19

Pur bump de version, aucun changement de code — publié pour qu'une
horloge fraîchement flashée en USB sur v87 ait une cible OTA réelle,
exerçant ainsi pour de vrai le buffer TLS 8192 ajouté en v87.

## v87 — beta uniquement — 2026-09-19

Teste un buffer de réception TLS de 8192 (au lieu de 16384). Le
diagnostic ajouté en v85 a confirmé que 16384 échoue par fragmentation
mémoire (heap=23272 total mais maxblk=15688, insuffisant pour
l'allocation ~16,7 Ko requise) — 8192 reste le double de l'original
4096 tout en laissant une marge confortable sous les 15688 observés.
Publié en beta/ uniquement.

## v86 — beta uniquement — 2026-09-19

Pur bump de version, aucun changement de code — publié pour qu'une
horloge fraîchement flashée en USB sur v85 ait une cible OTA réelle,
exerçant ainsi pour de vrai le diagnostic heap/fragmentation
(heap=/maxblk=) ajouté en v85.

## v85 — beta uniquement — 2026-09-19

Le buffer TLS 16384 (v83) a échoué 3/3 fois avec "connection failed" à
0 octet — pistes possibles : allocation ~16 Ko contiguë qui échoue sur
un tas fragmenté, ou tout autre cause. Ajoute au diagnostic
lastOtaPreConnectHeap/MaxBlock (ESP.getFreeHeap()/getMaxFreeBlockSize()),
capturés juste avant l'appel de connexion, pour objectiver la cause
avant de choisir la prochaine taille de buffer. Buffer TLS conservé à
16384 pour ce test. Publié en beta/ uniquement.

## v84 — beta uniquement — 2026-09-19

Pur bump de version, aucun changement de code — publié pour qu'une
horloge fraîchement flashée en USB sur v83 ait une cible OTA réelle,
exerçant ainsi pour de vrai le buffer TLS élargi à 16384 ajouté en v83.

## v83 — beta uniquement — 2026-09-19

Teste la piste la plus solide à ce jour : remonte le buffer de réception
TLS de 4096 à 16384 octets (la taille maximale d'un enregistrement TLS,
et le "minimum sûr" que les auteurs de la librairie BearSSL utilisent
eux-mêmes par défaut). Motivé par deux échecs indépendants s'arrêtant
au même octet exact (16355, à 29 octets de 16384) sur v81/v82. IMPORTANT :
comme tout changement du mécanisme de téléchargement OTA lui-même, doit
être flashé en USB avant qu'un test OTA ne soit valide (l'horloge qui
tente le téléchargement exécute son firmware ACTUEL pendant la
tentative, pas la cible visée). Publié en beta/ uniquement.

## v82 — beta uniquement — 2026-09-19

Pur bump de version, aucun changement de code — publié pour qu'une
horloge fraîchement flashée en USB sur v81 ait une cible OTA réelle,
exerçant ainsi pour de vrai la correction clone() ajoutée en v81 (le
comptage exact d'octets devrait enfin apparaître correctement si un
échec survient).

## v81 — beta uniquement — 2026-09-19

Corrige `exact=0` malgré des octets réellement transférés (61440
confirmés écrits en flash sur l'échec précédent). Cause : HTTPClient::
begin() ne garde jamais l'objet client passé en argument, il appelle
client.clone() et travaille avec ce clone pour toute la connexion —
WiFiClientSecure::clone() fait `new WiFiClientSecure(*this)`, codé en
dur sur le type de base, donc même appelé sur CountingWiFiClientSecure
(v77) il "tranchait" l'objet et produisait un client nu sans le
compteur. Fix : clone() surchargé pour préserver la sous-classe. Publié
en beta/ uniquement.

## v80 — beta uniquement — 2026-09-19

Pur bump de version, aucun changement de code — publié pour qu'une
horloge fraîchement flashée en USB sur v79 ait une cible OTA réelle,
exerçant ainsi pour de vrai la correction de troncature du diagnostic
ajoutée en v79.

## v79 — beta uniquement — 2026-09-19

Corrige la troncature silencieuse du message d'erreur OTA : deux échecs
réels sur une horloge confirmée en v77 ont remonté `... @X/Yb` sans le
`exact=...` ajouté en v77, alors que le code l'ajoutait bien. Cause
probable : la chaîne de concaténations `String` (jusqu'à 7 allocations
temporaires) peut échouer silencieusement sur un tas fragmenté juste
après la fermeture d'une session TLS ratée — `String::concat()`
abandonne sans erreur. Remplacé par un seul `snprintf` dans un buffer
fixe (une seule allocation finale). Publié en beta/ uniquement.

## v78 — beta uniquement — 2026-09-19

Pur bump de version, aucun changement de code — publié pour qu'une
horloge fraîchement flashée en USB sur v77 ait une cible OTA réelle,
exerçant ainsi pour de vrai le comptage exact d'octets ajouté en v77.

## v77 — beta uniquement — 2026-09-19

Ajoute un comptage exact des octets reçus (`CountingWiFiClientSecure`,
sous-classe de `WiFiClientSecure` qui compte dans `read()`), en
complément de `lastOtaProgressBytes` (v74) qui s'est révélé arrondi au
dernier bloc flash de 4096 octets confirmé écrit (voir
`Updater::progress()`), pas la position réelle de coupure. Message
d'erreur désormais du type
`... @32768/533328b exact=35120`. Publié en beta/ uniquement.

## v76 — beta uniquement — 2026-09-19

Pur bump de version, aucun changement de code — l'horloge de test étant
déjà passée en v75 via OTA, elle se considérait "à jour" par rapport au
canal beta et ne redéclenchait plus l'écran de mise à jour au boot. v76
lui redonne une cible pour continuer à exercer le diagnostic OTA. Publié
en beta/ uniquement, racine et stable restent à v73.

## v74/v75 — rétrogradées en beta uniquement — 2026-09-19

v74 et v75 avaient été publiées directement en stable (racine + stable/ +
beta/, voir les deux entrées ci-dessous) au moment où elles ne
contenaient qu'un ajout de diagnostic à faible risque. Une fois utilisées
pour un vrai test OTA, elles se sont révélées un terrain de test actif
pour le problème Stream Read Timeout non résolu (voir project_ota_update
memory) — décision de l'utilisateur : ne pas exposer le reste de la
flotte (canal stable) à des versions encore en cours d'investigation.
`racine/` et `stable/` restaurés au binaire v73 (dernier état validé,
commit `98250a2`) ; `beta/` conservé à v75. Toute nouvelle version tant
que ce chantier est ouvert sera publiée en beta uniquement, jusqu'à
décision conjointe de promotion en stable.

## v75 — stable (racine + stable/ + beta/) — 2026-09-19

Pur bump de version, aucun changement de code — publié uniquement pour
qu'une horloge fraîchement flashée en USB sur v74 ait une cible réelle
vers laquelle tenter un OTA, et exerce ainsi pour de vrai le diagnostic
de position (octets) ajouté en v74.

## v74 — stable (racine + stable/ + beta/) — 2026-09-19

Ajoute au check-in de flotte la position atteinte (octets reçus/attendus)
au moment d'un échec OTA, en complément du message d'erreur seul. Sert à
objectiver l'hypothèse "fenêtre d'exposition" derrière les échecs
persistants Stream Read Timeout / connection lost (voir
project_ota_update memory) — ne change rien au mécanisme de
téléchargement lui-même, donc aucune amélioration de fiabilité attendue
avant la prochaine version. Publié directement en stable, pas de phase
beta.

## v73 — stable (racine + stable/ + beta/) — 2026-09-06

Persiste en EEPROM le timestamp d'entrée en mode nuit (ajouté en v72,
qui était en RAM uniquement et se perdait à chaque redémarrage). Publié
directement en stable, pas de phase beta.

## v72 — stable (racine + stable/ + beta/) — 2026-09-05

Ajoute le timestamp de dernière entrée en mode nuit au check-in de
flotte, pour que le dashboard sache si une horloge est passée en mode
nuit dans les dernières 24h (pas seulement son état instantané au
dernier check-in). Publié directement en stable, pas de phase beta.

## v71 — stable (racine + stable/ + beta/) — 2026-08-30

Corrige le clignotement de secours de la transition "Serpent (jeu)"
quand aucun chemin auto-évitant n'est trouvable (grille trop remplie) :
bascule sur la transition "Serpent" simple plutôt que d'afficher le
résultat final instantanément. Publié directement en stable, pas de
phase beta.

## v70 — stable (racine + stable/ + beta/) — 2026-08-30

Vérifie les mises à jour toutes les ~3h au lieu d'1x/jour (plus de
fenêtres de tentatives indépendantes face à un réseau instable), et
remonte au dashboard fleet un échec de simple vérification de version
(auparavant invisible). Ne change pas le mécanisme de téléchargement
lui-même. Publié directement en stable, pas de phase beta.

## v69 — stable (racine + stable/ + beta/) — 2026-08-29

Affiche la version firmware et l'ID de puce dans le portail de
configuration de l'horloge. Changement d'UI mineur, publié directement
en stable sur les trois emplacements, pas de phase beta.

## v68 — stable (racine + stable/ + beta/) — 2026-08-29

Introduit la configuration à distance (transition, bornes LDR, canal
OTA) et le mécanisme de canaux lui-même. Publié directement en stable
sur les trois emplacements (racine, `stable/`, `beta/`) — pas de phase
beta pour cette version fondatrice, puisqu'elle met en place le
mécanisme de canaux avant qu'il ne serve à quoi que ce soit.
