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
