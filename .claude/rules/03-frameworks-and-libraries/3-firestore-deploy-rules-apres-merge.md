---
description: Toute PR qui modifie `firestore.rules` DOIT être suivie d'un déploiement des rules — sinon PERMISSION_DENIED systématique en prod. Gérer aussi le conflit 409 sur `firestore.indexes.json`.
globs:
  - "firestore.rules"
  - "firestore.indexes.json"
alwaysApply: false
---

# Firestore — déployer les rules après merge d'une PR qui les modifie

## Pourquoi

Une PR qui ajoute/étend une règle dans `firestore.rules` (ou un index
dans `firestore.indexes.json`) **ne change rien côté serveur** tant
qu'on n'a pas exécuté `firebase deploy`. Le code applicatif mergé
s'appuie alors sur une règle qui n'existe pas encore en prod
→ `PERMISSION_DENIED` systématique pour les nouveaux chemins, ou
`MISSING_INDEX` pour les nouvelles queries composites.

## Règle

Dès qu'une PR touche `firestore.rules` ou `firestore.indexes.json` :

1. **Après merge**, déployer immédiatement :
   ```bash
   firebase deploy --only firestore:rules,firestore:indexes --project <projectId>
   ```
2. Noter dans la description de la PR : « ⚠️ Déploiement rules/indexes
   requis après merge » (rappel pour le reviewer / le PO).
3. Si l'env Firestore est unique (un seul projet = prod), le déploiement
   = action sur prod → demander confirmation avant de lancer.

## Conflit 409 sur les indexes

Quand une query composite échoue côté client, Firebase met **un lien de
création d'index dans le message d'erreur**. Si quelqu'un clique ce
lien, l'index est créé côté serveur mais il n'est PAS dans
`firestore.indexes.json` (ou sous une forme légèrement différente). Le
`firebase deploy --only firestore:indexes` échoue alors avec
`HTTP Error: 409, index already exists`, **et l'échec global empêche
le déploiement des rules dans la même commande**.

Solution :
- Déployer **rules et indexes séparément** : d'abord
  `firebase deploy --only firestore:rules` (le crucial), puis
  `firebase deploy --only firestore:indexes` (peut échouer en 409 — OK,
  l'index est déjà là).
- Ne PAS utiliser `--force` sur les indexes (ça supprimerait les index
  non listés dans le fichier — destructif).

## Bypass acceptables

- PR qui ne touche QUE le code applicatif → pas de déploiement requis.
- Tests `fake_cloud_firestore` (ignorent rules + indexes).
