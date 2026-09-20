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

## 103.1.6 — beta uniquement — 2026-09-20 — remplace https.getString() (renvoyait 0 octet)

Cause trouvée grâce aux logs de 103.1.5 : "Tide sites: corps recu, 0
octets" malgré un code HTTP 200 -- https.getString() échoue
silencieusement sur cette requête précise (pas d'exception, juste une
chaîne vide). Remplacée par une lecture manuelle du flux (stream->
available()/read()), le même schéma déjà éprouvé et fiable dans
downloadAndFlashOta() pour l'OTA elle-même -- pas une nouvelle méthode,
celle déjà validée par des dizaines de téléchargements réels sur cette
horloge.

## 103.1.5 — beta uniquement — 2026-09-20 — diagnostics /tide_sites

103.1.4 toujours vide côté portail, cause inconnue (aucune des branches
de handleTideSites() ne logguait, sauf l'échec HTTP non-200). Ajoute un
log sur chaque chemin (WiFi non connecté, échec https.begin(), taille du
corps reçu, nombre de ports trouvés) -- aucun changement de logique,
uniquement pour localiser la cause réelle avant de corriger.

## 103.1.4 — beta uniquement — 2026-09-20 — /tide_sites sans ArduinoJson (103.1.3 insuffisant)

103.1.3 (tampon 16384) toujours insuffisant en pratique -- confirmé en
direct sur une horloge réelle via moniteur série (crash "Unhandled C++
exception: OOM" avec 8192, échec propre "NoMemory" avec 16384, aucune
amélioration). Remplace complètement l'usage d'ArduinoJson pour
/tide_sites par un parsing texte simple (recherche de sous-chaînes) --
seul le corps brut (~13 Ko) est gardé en mémoire, jamais de document
JSON parsé en entier. Logique validée hors-cible contre la vraie réponse
api-maree.fr/sites (133 sites, accents/apostrophes inclus) avant
publication : zéro erreur.

## 103.1.3 — beta uniquement — 2026-09-20 — corrige la liste des ports vide

Bug réel remonté par l'utilisateur (liste de ports vide dans le portail
après OTA vers 103.1.2). Cause trouvée par calcul (pas de matériel
disponible) : le tampon JSON de /tide_sites (8192 octets) était trop
petit pour la vraie réponse de api-maree.fr/sites (133 ports, confirmée
en direct), causant un echec NoMemory silencieux cote ESP8266. Tampon
porte a 16384 octets.

## 103.1.2 — beta uniquement — 2026-09-20 — clé api-maree.fr déplacée côté serveur

Correction suite à 103.1.1 : la clé API api-maree.fr ne vit plus sur
l'horloge (ni codée en dur, ni saisie sur le portail) -- elle se serait
retrouvée extractible par n'importe qui depuis firmware.bin, distribué
publiquement. Les requêtes passent maintenant par un proxy
horloge-sebsab-fleet (api/tide.js, variable d'environnement Vercel
TIDE_API_KEY, jamais exposée à aucun client), même principe que
SUPABASE_SERVICE_ROLE_KEY déjà utilisée dans ce projet. Champ "Clé API"
retiré du portail ; seul le choix du port reste à configurer. EEPROM_SIZE
réduit (226-265 abandonnées, jamais réutilisées -- ce champ n'a jamais
tourné sur une horloge réelle).

## 103.1.1 — beta uniquement — 2026-09-20 — module marées (préparation, sans le déclencheur physique)

Nouvelle fonctionnalité en cours : affichage de la phase de marée sur la
grille de LED (jauge par lignes bleu/jaune puis fondu vers l'heure de la
prochaine pleine mer), destinée à être déclenchée par un double-tap
physique (accéléromètre ADXL345, pas encore posé). Ce qui est déjà là :
sélection du port suivi + clé API api-maree.fr depuis le portail (nouvelle
section "Marées"), récupération périodique (12h) des horaires PM/BM,
bouton "Aperçu marée" pour tester sans le capteur. Schéma JSON de
api-maree.fr vérifié en direct (clé réelle, port Port-en-Bessin) avant
publication -- correspond exactement à ce qui est codé. Toujours pas
testé sur une horloge physique. Voir project_tide_module (mémoire projet)
pour le détail complet des décisions de conception.

## 103.1.0 — STABLE — 2026-09-19 — promotion en stable (root + stable + beta)

Promotion de la 103.0.1 (validée : 3 OTA réussies d'affilée via le
mécanisme resumable v101, transition + nouvelle logique de comparaison
x.y.z toutes deux confirmées en direct). Convention respectée : MINOR
+1, PATCH remis à 0 (103.0.1 -> 103.1.0) pour marquer le passage en
stable, aucun changement de code au-delà du numéro de version. Publié
sur les trois canaux (root/, stable/, beta/) puisqu'il n'y a plus de
divergence active entre beta et stable à ce stade. Fin de la règle
"beta uniquement" qui protégeait la flotte pendant cette investigation
OTA — la flotte peut de nouveau recevoir des mises à jour stables
normalement.

## 103.0.1 — beta uniquement — 2026-09-19 — bump pur, aucun changement de code

Cible de test pour valider la NOUVELLE logique de comparaison par tuple
(103.0.0 → 103.0.0 avait seulement validé le parsing `toInt()` de
l'ANCIEN firmware ; ce bump teste le code de fetch/comparaison x.y.z
réécrit lui-même, une fois qu'une horloge tourne déjà en 103.0.0).

## 103.0.0 — beta uniquement — 2026-09-19 — NOUVELLE NUMÉROTATION x.y.z

À partir d'ici, numérotation MAJOR.MINOR.PATCH au lieu d'un entier
plat : **PATCH=0 signifie "version stable"**, PATCH≥1 signifie "N-ième
itération beta depuis la dernière stable". MINOR n'avance qu'au moment
de promouvoir une beta validée en stable (PATCH revient à 0). MAJOR
réservé aux changements d'architecture majeurs. Démarré à 103 (juste
après le dernier entier plat publié, v102) pour qu'une horloge encore
sur l'ancien schéma détecte correctement cette version comme plus
récente (elle compare de simples entiers, "103.0.0".toInt() = 103) sans
aucune modification de son côté. version.txt contient désormais la
chaîne complète "x.y.z", plus un entier seul. Test de la mécanique
elle-même avant toute promotion : transition ancien->nouveau schéma,
stockage EEPROM (nouvelles adresses 192/193 pour minor/patch), et
intégration dashboard (fw_version stocké en texte, migration Supabase
migration_009_fw_version_text.sql).

## v102 — beta uniquement — 2026-09-19

Pur bump de version, aucun changement de code — publié pour qu'une
horloge fraîchement flashée en USB sur v101 ait une cible OTA réelle,
exerçant ainsi pour de vrai le téléchargement reprenable (HTTP Range)
ajouté en v101 sur un vrai transfert.

## v101 — beta uniquement — 2026-09-19

Changement de stratégie majeur : après avoir écarté quatre causes
possibles (buffer TLS, hébergement/CDN, lecture octet-par-octet,
blocage flash pendant Update.write()) sans trouver la vraie cause du
"Stream Read Timeout" récurrent, le téléchargement OTA devient
reprenable au lieu de tout perdre à chaque coupure. GitHub supporte les
requêtes HTTP Range (vérifié) : sur un timeout, rouvre une connexion
avec `Range: bytes=<déjà reçu>-` et continue à écrire dans la même
session Update, au lieu d'abandonner tout le téléchargement. Budget de
10 minutes / 40 reprises max pour ne jamais bloquer indéfiniment. Les
coupures se produisant presque toujours entre ~15 et ~50 Ko sur un
fichier de ~530 Ko, quelques segments devraient statistiquement
suffire à compléter le fichier. Ne résout pas la cause profonde
(toujours inconnue) mais vise à rendre les mises à jour fiables malgré
elle.

## v100 — beta uniquement — 2026-09-19

Pur bump de version, aucun changement de code — publié pour qu'une
horloge fraîchement flashée en USB sur v99 ait une cible OTA réelle,
exerçant ainsi pour de vrai la mesure du temps Update.write() ajoutée
en v99.

## v99 — beta uniquement — 2026-09-19

Ajoute la mesure du temps passé dans Update.write() (cumulé + durée max
d'un appel) au diagnostic. Motivé par un vrai pattern observé en direct
sur cette session : la plupart des "Stream Read Timeout" se regroupent
entre 15400 et 15460 octets, proche d'une fenêtre de congestion TCP
initiale typique. Hypothèse à confirmer : l'effacement/écriture flash
déclenché par Update.write() tous les 4096 octets pourrait tourner
interruptions désactivées (comportement SDK ESP8266) et priver la pile
WiFi/TCP assez longtemps pour empêcher l'accusé de réception à temps.

## v98 — beta uniquement — 2026-09-19

Pur bump de version, aucun changement de code — publié pour qu'une
horloge fraîchement flashée en USB sur v97 ait une cible OTA réelle,
exerçant ainsi pour de vrai le correctif du crash getStreamPtr() ajouté
en v97.

## v97 — beta uniquement — 2026-09-19

Corrige un vrai crash trouvé en direct (session série + addr2line sur
epc1) : Exception (28) LoadProhibitedCause, excvaddr=0x00000000, dans
downloadAndFlashOta(). HTTPClient::getStreamPtr() peut renvoyer nullptr
si la connexion se referme entre GET() et cet appel — jamais vérifié
avant, provoquant un appel de méthode virtuelle sur pointeur null
(lecture à l'adresse 0). Ajoute la vérification manquante. N'explique
pas le Stream Read Timeout lui-même, mais élimine un vrai plantage
silencieux (reboot sans aucun message d'erreur) rencontré pendant
l'investigation.

## v96 — beta uniquement — 2026-09-19

Pur bump de version, aucun changement de code — publié pour qu'une
horloge fraîchement flashée en USB sur v95 ait une cible OTA réelle,
exerçant ainsi pour de vrai le diagnostic RSSI/canal WiFi ajouté en
v94 (conservé dans v95) sur un vrai échec.

## v95 — beta uniquement — 2026-09-19 — REPRISE DES PUBLICATIONS ICI

Le canal beta revient à GitHub (racine + stable inchangés pendant tout
ce temps) : le test Vercel (v90-94) n'a pas identifié le CDN GitHub/
Fastly comme cause du blocage déterministe récurrent — nouveau point
de blocage tout aussi déterministe sur Vercel, pas d'amélioration.
Infra Vercel (firmware/, vercel.json, api/clocks.js) nettoyée côté
dépôt firmware-releases fleet. Publications beta reprennent ici
normalement à partir de cette version.

## v90 — beta uniquement — 2026-09-19 — DERNIÈRE VERSION PUBLIÉE ICI PENDANT LE TEST

Route le canal beta vers l'infra Vercel/Supabase existante
(clocksebsab-fleet.vercel.app/firmware/beta/) au lieu de
raw.githubusercontent.com, pour tester si le CDN GitHub/Fastly est en
cause dans les coupures persistantes (ni 4096, ni 16384, ni 8192
n'ont déplacé le point de blocage récurrent à l'octet 16355). Stable
reste inchangé sur GitHub. v90 est publiée ici comme d'habitude car
l'horloge de test (v89) lit encore GitHub pour ce dernier saut — une
fois v90 confirmée installée, les publications suivantes du canal beta
se feront uniquement sur clocksebsab-fleet.vercel.app/firmware/beta/
(voir ce dépôt), pas ici, tant que ce test est en cours.

## v89 — beta uniquement — 2026-09-19

Clôt la piste "buffer TLS trop petit" (v82-v88) : trois configurations
différentes (4096, 16384, 8192) ont produit le même point de coupure
exact (octet 16355) — le point de blocage ne bougeant pas avec la
taille du buffer, ce n'en est vraisemblablement pas la cause. Retour à
4096 (valeur d'origine). Tout l'outillage de diagnostic construit
pendant cette investigation (comptage exact, heap/fragmentation) est
conservé. Publié en beta/ uniquement.

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
