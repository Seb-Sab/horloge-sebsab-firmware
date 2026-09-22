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

## 104.0.1 — beta uniquement — 2026-09-22 — fix couleur jauge marée descendante

Corrige un bug signalé par l'utilisateur dans `showTideDisplay()` : en
marée descendante, les lignes qui grandissaient au fil du temps étaient
BLEUES au lieu de JAUNES (le comportement voulu, documenté en commentaire
depuis 103.1.30, n'était pas ce que le code produisait réellement).
L'eau (bleu) occupe maintenant toujours les lignes du bas de la jauge,
montante ou descendante ; seul l'ordre de révélation change selon la
direction. Publiée en beta uniquement, en attente de confirmation
visuelle sur une horloge réelle avant promotion en stable.

## 104.0.0 — stable (+ beta) — 2026-09-21 — première stable du module marées

Promotion en stable du lot validé en beta de 103.1.23 à 103.1.55 (voir
les entrées 103.1.53 à 103.1.55 ci-dessous pour le détail). Publiée sur
les trois emplacements (racine, `stable/`, `beta/`) pour qu'aucune
horloge, quel que soit son canal, ne reste bloquée derrière.

Numéro de version : MAJOR avancé (103 → 104) à la demande explicite de
l'utilisateur pour marquer l'intégration du mode marée comme version
majeure, au lieu du 103.2.0 que donnait la convention MINOR+1 — la
comparaison est en tuple, donc toute horloge en 103.x la voit bien comme
plus récente. Le mécanisme OTA a été revalidé de bout en bout avant cette
promotion (103.1.55 installée sans reboucler, grâce au correctif de
`Update.write()` de 103.1.54).

Contenu : affichage marée (jauge LED animée, heure de la prochaine
marée pertinente à la direction, couleurs bleu/jaune, fondus), déclenchement
physique par double-tap (ADXL345, détecté à l'exécution : une horloge sans
le capteur fonctionne normalement, seule la section "Marées" du portail est
alors masquée), et remontée `adxl_present`/`tide_site` au check-in de flotte.

## 103.1.55 — beta uniquement — 2026-09-21 — OTA de test (aucun changement fonctionnel)

Binaire identique a 103.1.54, seul le numero de version change. Sert a
verifier que le correctif OTA de 103.1.54 (verification de la valeur de
retour de `Update.write()`) fonctionne reellement de bout en bout :
une horloge deja sur 103.1.54 doit telecharger, appliquer et redemarrer
sur 103.1.55 sans reboucler sur un nouveau telechargement.

## 103.1.54 — beta uniquement — 2026-09-21 — verifie enfin la valeur de retour de Update.write()

Symptome rapporte en direct : une mise a jour OTA se telechargeait
completement (barre de progression/logs jusqu'au bout) mais semblait ne
jamais s'appliquer -- au demarrage suivant, une nouvelle mise a jour etait
retelechargee, en boucle. Cause trouvee en lisant le vrai code source
d'Updater.cpp (framework) : `Update.write()` peut renvoyer MOINS d'octets
que demande (echec silencieux d'effacement/ecriture flash cote framework,
`_writeBuffer()`), mais sa valeur de retour n'etait jamais verifiee ici --
`written` avancait sur la base des octets RECUS du reseau, pas des octets
REELLEMENT ecrits en flash. Fix : ne compte desormais que les octets
reellement ecrits, et retente ce segment (meme mecanisme de reprise que
pour un echec reseau) si Update.write() en a ecrit moins que demande.

## 103.1.53 — beta uniquement — 2026-09-20 — module marées complet de bout en bout (affichage + double-tap physique)

Lot consolidé couvrant 103.1.23 → 103.1.53 (une seule entrée, pas 31 —
voir git log pour le détail commit par commit). Deux volets :

**Affichage marées** (jauge LED + heure de prochaine marée) : fetch
`fetchTideExtrema()` corrigé (synchro NTP déplacée avant l'appel, lecture
bufferisée manuelle au lieu de `HTTPClient`/`ArduinoJson`, tampon BearSSL
agrandi, buffers réseau libérés avant le parse JSON) ; jauge révélée ligne
par ligne avec fondu (bas→haut si montante, haut→bas si descendante),
scintillement de nuances de bleu sur les lignes immergées, heure affichée
= prochaine marée pertinente à la direction (haute si montante, basse si
descendante) colorée bleu/jaune, fondu retour à l'heure réelle en fin de
séquence. Un bug de dépassement de pile (`playTransitionFade()` avec 4
tableaux `NUM_LEDS` simultanés dans le contexte `webServer.on()`) corrigé
au passage.

**Déclenchement physique (ADXL345, double-tap)** : driver I2C écrit à la
main (pas de librairie), sonde de présence au boot (deux adresses
possibles), configuration du double-tap. Cause racine d'un long
diagnostic de câblage : le silkscreen D3/D4 de cette carte physique ne
correspond pas à la table de broches "d1" compilée par `platformio.ini`
(GPIO réels 0/2, convention "d1_mini"/NodeMCU) — déterminé empiriquement
par balayage GPIO. `THRESH_TAP` calibré à ~1.2g après plusieurs essais en
direct ; cooldown logiciel porté à 9s (> durée de l'affichage) pour éviter
un rebond mécanique pendant l'affichage qui redéclenchait aussitôt.

Section "Marées" du portail masquée entièrement si le capteur ADXL345
n'est pas détecté sur l'horloge (`adxlPresent` dans `/config`).

## 103.1.22 — beta uniquement — 2026-09-20 — accumulation directe dans tideSitesJson (évite les réallocations de body)

103.1.21 a bien détecté l'échec cette fois (sa vérification a
fonctionné), mais confirmé en direct sur 5 tentatives consécutives :
la copie était tronquée de façon **variable** à chaque fois (5056,
5033, 3520, 4032, 2496 octets sur 6057, jamais le même nombre) — pas
un problème de capacité simple, puisque `tideSitesJson.reserve(6200)`
était déjà appelé tout au début de `setup()`. Cause probable :
`String::changeBuffer()` (code source du core ESP8266) n'arrondit qu'au
multiple de 16 le plus proche de la taille demandée — pas de croissance
exponentielle — donc chaque `concat()` sur la String locale `body` qui
dépassait sa capacité courante déclenchait sa propre réallocation
indépendante pendant la lecture (une douzaine au total pour ~6 Ko lus
par blocs de 512 octets), chacune pouvant échouer selon l'état du tas
à cet instant précis, ce qui explique la variabilité observée. Fix :
accumulation directe dans `tideSitesJson` (déjà réservé), plus de
variable `body` intermédiaire ni de copie finale — plus aucune
réallocation nécessaire pendant la boucle de lecture.

## 103.1.21 — beta uniquement — 2026-09-20 — vérifie/réserve la copie de tideSitesJson (échec silencieux du tas)

Le diagnostic ajouté en 103.1.20 ("Tide sites: envoye X/Y octets") a
révélé "0/0" à chaque requête, alors même que "corps recu 6057/6057"
et "liste chargee" venaient de s'afficher juste avant, dans le même
boot — donc pas une histoire de troncature réseau ou d'API de
sérveur web, mais une donnée réellement vide côté ESP. Cause :
`String::operator=` peut échouer silencieusement sur Arduino si le
tas est trop fragmenté pour trouver un bloc contigu de la taille
voulue (~6 Ko, non négligeable face aux ~28 Ko de tas libre typiques à
ce stade du boot) — la String de destination reste alors vide, sans
exception ni erreur. Rien ne vérifiait le résultat de
`tideSitesJson = body;`, donc "liste chargee" s'affichait quand même,
masquant l'échec. Fix : vérifie explicitement la longueur après la
copie (si incorrecte, `tideSitesFetched` reste faux, `loop()`
retentera dans 30s) ; réserve aussi `tideSitesJson` (6200 octets) le
plus tôt possible dans `setup()`, avant toute autre allocation
WiFi/EEPROM/webServer, pour réclamer le bloc contigu quand le tas est
encore frais.

## 103.1.20 — beta uniquement — 2026-09-20 — écriture manuelle sur le client TCP brut pour /tide_sites

103.1.19 (découpage en morceaux de 512 octets avec yield() entre
chacun) a été testé en direct : même symptôme exact,
`SyntaxError: Unexpected end of JSON input` sur 20/20 tentatives. Le
découpage n'a rien changé car `webServer.sendContent()` renvoie
`void` — impossible de détecter un envoi partiel même avec cette API
en petits morceaux. Fix : écriture manuelle directe sur
`webServer.client()` (le `WiFiClient` brut), en-têtes HTTP construits
à la main, corps envoyé via `client.write()` avec vérification
explicite du nombre d'octets réellement écrits à chaque appel et
reprise à la bonne position sinon — même principe déjà appliqué à la
lecture réseau dans ce module, appliqué ici à l'écriture. Ajoute un
log série du nombre d'octets réellement envoyés pour confirmer ou
infirmer en direct.

## 103.1.19 — beta uniquement — 2026-09-20 — envoie /tide_sites en petits morceaux (yield entre chaque)

Diagnostic définitif obtenu via le `console.error()` ajouté en
103.1.18 : `SyntaxError: Failed to execute 'json' on 'Response':
Unexpected end of JSON input` sur les 20/20 tentatives, malgré un
statut 200 — la réponse arrivait **tronquée côté navigateur**, cette
fois sur le trajet ESP→navigateur (pas ESP→Vercel). Cause trouvée dans
le code source d'ESP8266WebServer : `send(code, type, String)` envoie
tout le corps en un seul appel bloquant à `Stream::sendSize()` (voir
`ESP8266WebServer-impl.h::sendContent()`), qui peut renvoyer moins
d'octets que demandé après un timeout d'écriture — silencieusement
pour l'appelant. Pour ~6 Ko d'un coup, ça correspond exactement au
symptôme. Fix : `setContentLength()` + `send(200, type, "")` pour
n'envoyer que les en-têtes, puis plusieurs `sendContent()` de 512
octets avec `yield()`+`delay(1)` entre chacun — même principe déjà
appliqué à la lecture réseau tout au long de ce module.

## 103.1.18 — beta uniquement — 2026-09-20 — force cache:no-store sur /tide_sites

103.1.17 a corrigé le timing des réessais, mais un nouveau test a
montré 20/20 tentatives renvoyant un statut 200 (confirmé via l'onglet
Réseau du navigateur) alors que le serveur confirmait avoir la liste
en cache (log `6057/6057 octets` / `liste chargee`) — le menu
déroulant restait pourtant vide en permanence. Hypothèse : le
navigateur re-servait une réponse mise en cache tôt (le `[]` d'un
essai avant que le fetch serveur n'ait abouti) sans jamais retaper
réellement le réseau, malgré un 200 affiché à chaque "tentative".
Fix côté client : `fetch('/tide_sites', {cache:'no-store'})`. Fix côté
serveur en complément : en-tête `Cache-Control: no-store` explicite
sur cette réponse. Ajoute aussi un `console.error()` dans le portail
pour révéler la vraie exception si cette hypothèse s'avère fausse au
prochain test.

## 103.1.17 — beta uniquement — 2026-09-20 — élargit la fenêtre de réessai du portail pour /tide_sites

103.1.16 a corrigé le fetch réseau lui-même (confirmé en direct, deux
succès consécutifs) mais un nouveau symptôme est apparu côté portail :
après avoir sélectionné un port et sauvegardé, le menu déroulant
apparaissait vide au redémarrage ET la sélection n'était pas
mémorisée. Cause : au démarrage, l'horloge enchaîne 4 appels HTTPS
séquentiels dans `setup()` (vérification MAJ, check-in de flotte,
horaires de marée, puis la liste des ports elle-même) avant que
`/tide_sites` ne réponde autre chose que `503` — confirmé en direct,
ce cycle peut prendre 10-20s. L'ancienne fenêtre de réessai côté
portail (3 essais × 1,5s = 4,5s) abandonnait bien avant, laissant le
`<select>` avec seulement l'option par défaut — et l'assignation de
la sélection sauvegardée (qui a besoin d'une `<option>` déjà présente
pour fonctionner) échouait donc silencieusement aussi, donnant
l'illusion d'un port "non mémorisé" alors que l'EEPROM était
vraisemblablement correcte. Passé à 20 essais × 1,5s (30s), avec un
message "Chargement…" affiché pendant les tentatives.

## 103.1.16 — beta uniquement — 2026-09-20 — contourne HTTPClient::getStreamPtr() + agrandit le buffer BearSSL RX

Le diagnostic ajouté en 103.1.14 a enfin montré le mécanisme précis
de l'échec `/tide_sites` : `HTTPClient::getStreamPtr()` (code source
de la lib) fait `if(connected()) return _client.get(); return
nullptr;`, et le log montrait `connected=0` juste après un `GET()`
200 avec pourtant un `Content-Length` correctement lu (6057, identique
à curl). `getStreamPtr()` renvoyait donc `nullptr`, et la boucle de
lecture n'était **jamais exécutée** — pas un problème de timing ni de
pile, juste ce garde-fou. Fix : lecture directe sur l'objet
`BearSSL::WiFiClientSecure` local (déjà en scope) au lieu de passer
par `getStreamPtr()`. Deuxième mode d'échec observé séparément
(`connected=1` mais 0 octet reçu quand même après 15s) : agrandit le
tampon de réception BearSSL de 1024 à 4096 octets pour cette fonction
uniquement (hypothèse : la réponse ~6 Ko, la plus grosse reçue par
cet ESP8266 hors OTA, dépasse la taille d'enregistrement TLS que le
tampon précédent pouvait contenir) — les autres appels HTTPS du
fichier (réponses bien plus petites) gardent leur tampon 1024.

## 103.1.15 — beta uniquement — 2026-09-20 — URGENT : revert CONT_STACKSIZE (cassait le boot)

103.1.14's `CONT_STACKSIZE=8192` empêchait l'horloge de démarrer :
plus aucune ligne "Firmware version" au moniteur série, boucle de
reset dès l'amorçage, confirmé en direct par l'utilisateur ("l'horloge
ne redemarre pas"). `cont_t s_cont` (qui embarque le tableau
`stack[CONT_STACKSIZE/4]`) est allouée sur la "SYS stack" du SDK
ESP8266 — une zone de taille FIXE déjà partagée avec le SDK lui-même
(WiFi, etc.), pas une pile extensible sans conséquence (voir le
commentaire au-dessus de `app_entry_redefinable()` dans
`core_esp8266_main.cpp`). Doubler à 8192 a dépassé ce budget avant
même `setup()`. Retour au défaut (4096) pour débloquer l'horloge
immédiatement. Le crash `cont_check`/Soft WDT du module marées reste
non résolu, à traiter autrement (voir `project_tide_module` en
mémoire) — ne pas ré-augmenter cette valeur sans d'abord vérifier la
taille réelle de la SYS stack disponible sur ce SDK/core.

## 103.1.14 — beta uniquement — 2026-09-20 — double CONT_STACKSIZE + diagnostic d'en-têtes /tide_sites

103.1.13 (fetch déplacé hors du callback webServer, donc plus de
nesting) a quand même reproduit le même crash `Exception (5)`
(addr2line → `cont_check`) qu'en 103.1.12 — preuve que le nesting
n'était pas (seul) en cause. Signature classique d'une pile de
continuation ESP8266 (~4 Ko par défaut) trop juste pour une poignée de
main BearSSL. Ajoute `build_flags = -D CONT_STACKSIZE=8192` dans
`platformio.ini` (vérifié applicable : les fichiers core concernés
sont recompilés à chaque build PlatformIO, pas une lib précompilée).
Séparément, le symptôme de base (`corps recu, 0/-1 octets`) persiste
même sans nesting ni crash — ajoute un log diagnostique
(Content-Length, Transfer-Encoding, Content-Encoding, Connection,
`getSize()`, `connected()`) sur la réponse réellement reçue par CET
ESP8266, pour trancher entre les hypothèses restantes au prochain test
plutôt que deviner un correctif de plus à l'aveugle.

## 103.1.13 — beta uniquement — 2026-09-20 — deplace le fetch /tide_sites hors du callback webServer

103.1.12 tentait de corriger un abandon prématuré de la lecture réseau
mais provoquait en fait de vrais crashs sur l'horloge réelle, à chaque
tentative d'ouverture du menu déroulant : `Exception (5)` (addr2line →
`cont_check`) une fois, puis `Soft WDT reset` + `Exception (4)`
(addr2line → `millis`) une autre fois — deux crashs distincts, tous
deux à des adresses génériques du framework, pas dans le code du
module. Cause réelle trouvée en comparant `handleTideSites()` aux
trois autres fonctions HTTPS de ce fichier (`fetchTideExtrema()`,
`checkForUpdate()`, `sendFleetCheckin()`) : c'était la SEULE à ouvrir
une connexion BearSSL depuis l'intérieur d'un callback
`webServer.on()` — les trois autres tournent toutes depuis
`setup()`/`loop()`. Piège ESP8266 connu : la pile de continuation
utilisée par les callbacks est petite (~4 Ko par défaut), et imbriquer
une poignée de main BearSSL (gourmande en pile) par-dessus la pile
déjà entamée par `webServer.handleClient()` peut la dépasser —
silencieusement ou via ce type de crash. Fix : plus aucun appel réseau
dans le handler HTTP. `fetchTideSitesList()` (nouvelle fonction, tourne
depuis `setup()` une fois puis `loop()` toutes les 30s tant que non
réussie) fait le travail réseau dans le même contexte sûr que les
autres appels HTTPS du fichier, et remplit un cache (`tideSitesJson`).
`handleTideSites()` relaie maintenant ce cache instantanément, sans
ouvrir la moindre connexion — en bonus, le menu déroulant du portail
se charge désormais sans latence réseau à l'ouverture.

## 103.1.12 — beta uniquement — 2026-09-20 — corrige l'abandon premature de la lecture /tide_sites

103.1.11 déportait bien /sites sur Vercel (curl confirme un JSON propre,
133 ports, Content-Length: 6057), mais le portail montrait un menu
déroulant vide et le moniteur série affichait systématiquement
"corps recu, 0/-1 octets" / "reponse incomplete, ignoree", à chaque
tentative, y compris juste après un reset. Cause : `https.connected()`
répond `false` dès le tout premier sondage de la boucle de lecture,
avant même l'arrivée d'un seul octet du corps -- la boucle sortait
donc immédiatement sans avoir rien lu, alors que le serveur envoyait
bien les données. Ajoute une marge de 300ms avant de faire confiance à
ce signal (au lieu de couper au premier sondage). Remplace aussi la
validation stricte par comptage d'octets (Content-Length peu fiable
sur cet ESP8266 tout au long de ce module, cf. 103.1.9/103.1.10) par
une validation de forme du JSON (tableau bien formé) quand la taille
annoncée est inconnue.

## 103.1.11 — beta uniquement — 2026-09-20 — /tide_sites déporté sur Vercel

Après six itérations de correctifs côté ESP8266 sur la récupération de
/sites (tampon JSON, parsing texte, détection de coupure, résynchro-
nisation) sans jamais obtenir une liste fiable à 100% -- corruption de
contenu confirmée en direct même sur des transferts "complets" en
octets, et même problème confirmé sur la liaison horloge->navigateur en
local (ERR_CONTENT_LENGTH_MISMATCH) -- changement d'approche, proposé
par l'utilisateur : la récupération + le nettoyage tournent maintenant
côté Vercel (horloge-sebsab-fleet/api/tide_sites.js, connexion
serveur-à-serveur fiable, cache 12h), l'horloge relaie une réponse déjà
propre et compacte. Simplifie fortement handleTideSites() (plus de
parsing/resynchronisation côté firmware).

## 103.1.10 — beta uniquement — 2026-09-20 — détecte le désynchronisme site_id/site_name

103.1.9 recevait bien le corps complet (received==total confirmé à
chaque fois) mais ne trouvait qu'entre 78 et 91 ports sur 133 réels,
de façon variable à chaque tentative. Cause : une corruption ponctuelle
peut faire apparaître un guillemet au mauvais endroit, associant un
site_id au site_name d'une AUTRE entrée plus loin dans le document --
ce désynchronisme se propage ensuite à toute la suite (chaque paire
mal alignée fausse la recherche de la suivante), bien au-delà de la
seule entrée touchée au départ. site_name doit toujours suivre
immédiatement site_id (séparés de ~15 caractères fixes) ; un écart
plus grand est maintenant détecté et ce site_id est ignoré au lieu de
laisser le décalage continuer. Entrées de longueur déraisonnable aussi
rejetées (protection contre un JSON de sortie invalide). Revérifié
contre les vraies données (133/133, zéro erreur) avant publication.

## 103.1.9 — beta uniquement — 2026-09-20 — rejette les réponses /tide_sites incomplètes + réessai auto

Cause exacte trouvée via la console du navigateur (ERR_CONTENT_LENGTH_
MISMATCH) : la liaison réseau coupe parfois en cours de transfert (même
phénomène que côté OTA toute cette session), et le code précédent
servait quand même ce qu'il avait reçu -- résultat observé en direct :
entrées visiblement corrompues (nom tronqué en plein milieu, ou du texte
d'un autre endroit du document collé à la suite) ou liste simplement
incomplète (arrêt avant la fin de l'alphabet). handleTideSites()
compare désormais explicitement les octets reçus à la taille annoncée
(Content-Length) et rejette (502 vide) toute réponse incomplète plutôt
que de la servir telle quelle. Côté portail, la liste réessaie
automatiquement jusqu'à 3 fois avant d'abandonner.

## 103.1.8 — beta uniquement — 2026-09-20 — récupère ~570 octets de RAM (cause précise identifiée)

Mesure directe demandée par l'utilisateur : compilation isolée (git
worktree) de la version juste avant le module marées, comparaison
binaire section par section. Trouvé : ce n'est PAS les variables
globales du module marées (~250 octets seulement) mais l'usage de
strftime() dans fetchTideExtrema() qui importait toute la table de
formatage/locale de la libc (noms des mois/jours, plusieurs formats
date/heure -- jamais utilisée ailleurs dans ce fichier, qui formate
toujours les dates à la main), soit l'essentiel des ~1,3 Ko de RAM
supplémentaires constatés. Remplacé par snprintf() (déjà le pattern
utilisé partout ailleurs). Gain secondaire : DeserializationError::
c_str() (table de messages ArduinoJson) remplacé par le code d'erreur
numérique. RAM 48444->47872 (-572 octets). Toujours ~924 octets
au-dessus de la version d'avant module marées (structures/chaînes
propres au module, attendu et accepté).

## 103.1.7 — beta uniquement — 2026-09-20 — réduit les tampons BearSSL de l'OTA (crash OOM répété)

Crash reproductible confirmé 4 fois d'affilée en direct via moniteur
série (dump de pile complet, décodé à l'addr2line) : "Unhandled C++
exception: OOM" dans BearSSL::WiFiClientSecureCtx::WiFiClientSecureCtx()/
_connectSSL(), y compris juste après un reset physique (tas mémoire
garanti neuf) -- donc pas une fragmentation accumulée. Cause probable :
le module marées (103.1.1-103.1.6) a fait grossir la RAM statique
d'environ 1,6 Ko (57,3%->59,3%), suffisant pour faire basculer
l'allocation des tampons BearSSL de l'OTA (4096/1024, déjà limite) du
côté "échoue systématiquement". Réduit à 2048/512 dans
downloadAndFlashOta(). **Flash USB recommandé plutôt qu'OTA** pour cette
version précise : l'horloge encore en 103.1.5/103.1.6 utilise l'ANCIEN
code (gros tampons) pour tenter de télécharger cette correction --
situation potentiellement bloquante en boucle si le crash est
suffisamment systématique.

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
