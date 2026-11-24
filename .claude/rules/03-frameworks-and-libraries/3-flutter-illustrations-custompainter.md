---
description: APPLIQUER cette règle DÈS QU'un écran Flutter a besoin d'une illustration, d'un dessin, d'une scène ou d'un pictogramme décoratif (au lieu d'une image générée ou exportée).
globs: "**/*.dart"
alwaysApply: false
---

# Illustrations Flutter : dessinées avec CustomPainter

## Règle

- Toute illustration de l'interface (scène, décor, personnage, plante, objet, graphique décoratif, fond animé) est **dessinée en code avec `CustomPainter`**, jamais livrée en image générée (PNG/JPEG/WebP produits par une IA ou exportés d'un outil).
- Les dessins sont des **objets réutilisables et paramétrables** : un fichier par objet (ex. `SunPainter`, `PlantPainter`, `TubePainter`), assemblés par un widget de scène. Les paramètres décrivent l'état (taille, progression, humeur, heure, saison…) au lieu de dupliquer des variantes.
- Couleurs, rayons, épaisseurs et durées viennent des **jetons du thème** ; aucune valeur visuelle en dur.

## Fidélité à la maquette

- Reproduire la maquette **au détail près** : formes, proportions, couleurs, positions, épaisseurs, ombres, textes. Porter la géométrie de la source (SVG, HTML/CSS, fichier de design) plutôt que l'approximer.
- Vérifier par **comparaison visuelle côte à côte** : rendu Flutter (test golden ou capture `RepaintBoundary` à la même taille logique) et rendu de la maquette, en regardant les deux images, jusqu'à correspondance. Noter chaque écart restant et sa raison.

## Animation et accessibilité

- Animations dessinées dans le painter, pilotées par **une seule horloge partagée** par scène ; durées, courbes et amplitudes reprises des spécifications de la maquette.
- « Réduire les animations » (`MediaQuery.disableAnimationsOf`) : plus de mouvement continu (balancement, flottement, scintillement), seulement des fondus courts.
- La scène a un **libellé sémantique** qui décrit ce qu'elle montre ; les éléments purement décoratifs sont exclus de la sémantique.

## Performance

- `shouldRepaint` précis (comparer les paramètres), `RepaintBoundary` par couche, parties statiques enregistrées une fois (`PictureRecorder`) et seule la partie animée repeinte.
- Aucun calcul lourd par image ; pas de nouvelle allocation de `Paint`/`Path` évitable dans `paint`.

## Tests

- Tests unitaires de la géométrie (positions, tailles selon les paramètres) et tests golden des états clés (et en texte agrandi si la scène contient du texte).
- Les goldens sont générés sur une plateforme de référence documentée.

## Exceptions

- Photos réelles et contenu fourni par l'utilisateur.
- Icône d'application, écran de démarrage natif et assets imposés par la plateforme (formats raster requis) : générés à partir d'une source vectorielle versionnée et d'un script reproductible.
- Logo de marque fourni en vectoriel (SVG) par la charte.
