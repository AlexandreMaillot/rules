---
description: APPLIQUER ASCII-only sur les clés Firestore éditées par dot-notation `tx.update` LORS de la création/modification d'un document Firestore.
globs: apps/*/lib/**/*.dart, functions-src/**/*.ts, apps/*_api/**/*.dart
alwaysApply: true
---

Principe :
- Aucun caractère non-ASCII (accents `é è à ç`, ligatures, emoji, etc.) dans les **noms de champs** ou **noms de sous-objets** d'un doc Firestore qui sera édité via dot-notation `tx.update({'a.b.c': v})`.
- La regex `^[_a-zA-Z][_a-zA-Z0-9]*$` doit matcher chaque segment du chemin.

Pourquoi :
- Le SDK `dart_firebase_admin` (`FieldPath.fromArgument(String)`) split la clé sur `.` puis quote en backticks tout segment ne matchant pas la regex ASCII (cf. `lib/src/google_cloud_firestore/path.dart:_formattedName`). Les segments avec accent (`suppléments`) sont quotés et l'écriture `tx.update` peut être silencieusement perdue / partir dans une clé différente côté Firestore.
- Bug typique : une clé avec `é` casse un save sans aucune erreur côté serveur. Détection difficile. Fix : passer la clé en ASCII (`suppléments` → `supplements`).

Cas concrets :
- ✅ `suppléments` → `supplements`
- ✅ `tarifs_préfectoraux` → `tarifs_prefectoraux`
- ✅ `règles_métier` → `regles_metier`
- ✅ `créateur` → `createur`

Mitigation lecture des docs legacy :
- Pour les docs déjà écrits avec accents en prod, accepter les deux variantes côté lecteur en **préférant la variante ASCII** :
  ```dart
  final supplements = _asMap(map['supplements']) ?? _asMap(map['suppléments']);
  ```
- Documenter dans le code que la clé avec accent est legacy et que la clé ASCII est la source de vérité d'écriture.

Audit existant :
- `grep -rn "[é|è|à|ç|ê|î|ô|û|ï]" apps/*/lib/**/modeles/ apps/*_api/lib/` doit retourner uniquement des chaînes utilisateur (libellés), pas des **clés Firestore**.
- Vérifier en particulier les `factory fromFirestore`, `toFirestoreMap`, et toutes les routes de sauvegarde.

Note BDD :
- Les **valeurs** stockées peuvent contenir des accents sans problème (libellés multi-lignes, noms propres, etc.).
- Seules les **clés** (noms de champs / sous-objets) doivent rester ASCII.
