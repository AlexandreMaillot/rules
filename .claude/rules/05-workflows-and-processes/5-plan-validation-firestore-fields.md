---
description: Avant de rédiger un plan qui référence des collections/champs Firestore existants, grep le code pour valider les noms exacts (snake_case officiel) au lieu de supposer. Évite les corrections post-rédaction quand l'index composite ou la requête est construit.
globs: aidd_docs/tasks/**/*.md
alwaysApply: false
---
- Avant d'écrire un nom de champ Firestore dans un plan, `grep` le modèle Dart + `fromFirestore` pour récupérer la clé snake_case réelle.
- Idem pour les noms de collections : vérifier via `collection('nom')` dans les dépôts existants, ne pas inventer par traduction mécanique.
- Quand un index composite est planifié, copier-coller exactement la combinaison observée dans les requêtes existantes (ordre, ASC/DESC).
- Exemple de piège : un plan annonce `entite_id + type_prestation` alors que le code utilise `id_entite + prestation` → correction nécessaire au moment de construire l'index.
- Règle de validation minimale avant dépôt du plan : le plan ne doit contenir **aucun** nom de champ Firestore qui ne puisse être retrouvé par `grep` dans `apps/*/lib/**/modeles/` ou `apps/*/lib/**/depots/`.
