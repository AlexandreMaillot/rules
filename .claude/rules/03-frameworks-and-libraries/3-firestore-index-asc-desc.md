---
description: Firestore — un changement DESC↔ASC d'un `orderBy` nécessite un nouvel index composite à AJOUTER (pas remplacer).
globs:
  - "firestore.indexes.json"
  - "**/lib/**/depot*.dart"
  - "**/lib/**/service*firestore*.dart"
alwaysApply: false
---

# Firestore — index composite ASC vs DESC

## Pourquoi

Quand on bascule un `orderBy('champ', descending: true)` vers
`orderBy('champ')` (ASC), Firestore demande un **nouvel index
composite** dans la majorité des cas. Le moteur ne reverse-scan pas
toujours un index DESC pour servir une query ASC (composite avec
`__name__` notamment).

Conséquence en prod : `MISSING_INDEX` côté SDK + une URL de création
d'index dans la console Firebase. Erreur visible côté user.

## Règle

Quand on change l'`orderBy` d'un champ d'un dépôt Firestore :

1. **Ajouter** le nouvel index ASC/DESC dans `firestore.indexes.json`.
2. **Ne pas supprimer** l'ancien sauf certitude qu'aucune autre query
   ne l'utilise (tous les clients + server-side).
3. **Doubler la version SPARSE_ALL** si l'ancien index l'avait
   (avec `__name__` ASC/DESC en dernier champ).
4. **Déployer obligatoirement** : `firebase deploy --only firestore:indexes`
   AVANT le déploiement du code applicatif.

Exemple — passage de DESC à ASC :

```jsonc
// Avant (existant)
{
  "collectionGroup": "demandes",
  "fields": [
    { "fieldPath": "id_client", "order": "ASCENDING" },
    { "fieldPath": "heure_creneau", "order": "DESCENDING" }
  ]
}

// Après — on AJOUTE (sans supprimer l'ancien) :
{
  "collectionGroup": "demandes",
  "fields": [
    { "fieldPath": "id_client", "order": "ASCENDING" },
    { "fieldPath": "heure_creneau", "order": "ASCENDING" }
  ]
}
```

## Bypass acceptables

- Query single-field (Firestore reverse-scan automatique).
- `where(X) order by X` (single-field auto-indexé).
- Tests `fake_cloud_firestore` (ignorent les index).
