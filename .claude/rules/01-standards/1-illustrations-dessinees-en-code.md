---
description: APPLIQUER cette règle DÈS QU'une interface (mobile ou web) a besoin d'une illustration, d'un dessin, d'une scène ou d'un pictogramme décoratif : la dessiner en code vectoriel au lieu d'utiliser une image générée ou exportée.
alwaysApply: true
---

# Illustrations dessinées en code vectoriel

## Règle

- Toute illustration de l'interface (scène, décor, personnage, plante, objet, graphique décoratif, fond animé) est **dessinée en code vectoriel**, jamais livrée en image générée (PNG/JPEG/WebP produits par une IA ou exportés d'un outil).
- Technique selon la plateforme :
  - **Flutter** : `CustomPainter` (+ `Canvas`, `Path`).
  - **Web** : SVG en ligne ou composant SVG (React/Vue…), ou `<canvas>` pour les scènes très animées ; CSS pour les formes simples.
  - **Autres frameworks natifs** : l'API de dessin vectoriel de la plateforme (ex. Canvas Android, Core Graphics / SwiftUI `Shape`).
- Les dessins sont des **objets réutilisables et paramétrables** : un fichier par objet (ex. soleil, plante, décor), assemblés par une scène. Les paramètres décrivent l'état (taille, progression, humeur, heure, saison…) au lieu de dupliquer des variantes.
- Couleurs, rayons, épaisseurs et durées viennent des **jetons du design system / thème** ; aucune valeur visuelle en dur.

## Fidélité à la maquette

- Reproduire la maquette **au détail près** : formes, proportions, couleurs, positions, épaisseurs, ombres, textes. Porter la géométrie de la source (SVG, HTML/CSS, fichier de design) plutôt que l'approximer.
- Vérifier par **comparaison visuelle côte à côte** du rendu de l'app et du rendu de la maquette, à la même taille, en regardant les deux images, jusqu'à correspondance (test d'image de référence / golden, capture de composant, capture de navigateur headless). Noter chaque écart restant et sa raison.

## Animation et accessibilité

- Animations définies dans le dessin et pilotées par **une seule horloge** par scène ; durées, courbes et amplitudes reprises des spécifications de la maquette.
- Préférence « réduire les animations » du système (et de l'app si elle en a une) : plus de mouvement continu (balancement, flottement, scintillement), seulement des fondus courts. Web : `@media (prefers-reduced-motion: reduce)`.
- La scène a une **alternative textuelle** qui décrit ce qu'elle montre (libellé sémantique, `role="img"` + `aria-label`/`<title>`) ; les éléments purement décoratifs en sont exclus (`aria-hidden`).

## Performance

- Ne redessiner que ce qui change : couches isolées, parties statiques mises en cache, seule la partie animée recalculée à chaque image.
- Aucun calcul lourd ni allocation évitable par image.

## Tests

- Tests de la géométrie (positions, tailles selon les paramètres) et tests d'image de référence des états clés, générés sur une plateforme de référence documentée.

## Exceptions

- Photos réelles et contenu fourni par l'utilisateur.
- Icône d'application, écran de démarrage natif, favicon et images imposées par une plateforme ou un réseau social (formats raster requis) : générés à partir d'une source vectorielle versionnée et d'un script reproductible.
- Logo de marque fourni en vectoriel (SVG) par la charte.
