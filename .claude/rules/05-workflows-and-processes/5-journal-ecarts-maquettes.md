---
description: APPLIQUER cette règle DÈS QUE l'utilisateur demande un changement qui diffère des maquettes de design (ou un écran, état, texte ou animation sans maquette) : le consigner dans le journal des écarts pour l'exporter vers l'outil de design.
alwaysApply: true
---

# Journal des écarts aux maquettes

## Principe

- Les maquettes du projet (exports de l'outil de design, handoff, fichiers de design versionnés) font foi.
- Un écart n'est admis que s'il est **demandé par l'utilisateur**. Un écart imposé par une contrainte (accessibilité, contraste, sécurité, plateforme) est **signalé** à l'utilisateur, jamais décidé en silence.
- **Chaque demande de l'utilisateur qui diffère d'une maquette, ou qui ajoute un écran, un état, un texte ou une animation sans maquette, est consignée** dans un journal, pour qu'il puisse l'exporter vers l'outil de design et mettre les maquettes à jour.

## Où

- Un seul fichier de journal par projet. Son chemin est défini par la mémoire ou une règle du projet ; à défaut : `aidd_docs/design/journal-ecarts-maquettes.md` (le créer s'il n'existe pas).
- Le fichier contient :
  - en tête, une courte consigne pour l'outil de design (garder la direction visuelle existante, fournir les états, le texte agrandi et le mode animations réduites) ;
  - un tableau « Écarts demandés » pour les écrans déjà maquettés ;
  - des sections détaillées pour les nouveaux écrans ou états sans maquette (comportement, états, textes exacts, animations, règles).

## Quand

- **Au moment de la demande**, dans la conversation ou dans le lot qui l'implémente, sans attendre la fin du lot.
- Le commit qui implémente l'écart contient aussi la ligne du journal.
- Une relecture refuse un lot dont un écart demandé n'est pas consigné.

## Format d'une ligne

| Date | Écran | Maquette actuelle | Voulu par l'utilisateur |
|---|---|---|---|
| AAAA-MM-JJ | identifiant et nom de l'écran (ou « Partout ») | ce que montre la maquette, en une phrase | ce que l'utilisateur veut, précis et actionnable (textes exacts, valeurs, états) |

- Une ligne par décision. Une décision qui en remplace une autre **met à jour la ligne existante** (nouvelle date), sans laisser deux versions contradictoires.
- Les plans et specs de lot renvoient au journal au lieu de le dupliquer.

## Quand de nouvelles maquettes arrivent

- Quand l'utilisateur fournit un nouvel export qui intègre des écarts du journal, retirer ces lignes (elles sont devenues la maquette) et noter la nouvelle référence là où le projet liste ses maquettes.
