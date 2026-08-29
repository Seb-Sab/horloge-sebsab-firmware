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
