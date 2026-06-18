---
description: Pour notifier un utilisateur (FCM) ou lire ses fcm_tokens depuis le backend quand le doc.id Firestore N'EST PAS l'uid Auth, NE JAMAIS faire `collection/{uid}` — résoudre le doc via une requête `where uid == <uid>`.
globs:
  - "apps/*_api/lib/notifications/**/*.dart"
  - "apps/*_api/lib/push/**/*.dart"
alwaysApply: false
---

# FCM — résoudre le doc.id par `uid` quand doc.id ≠ uid Auth

## Pourquoi

Quand le token FCM est écrit sur **`<collection>/{docId}.fcm_tokens`** et que `docId` est un identifiant Firestore auto **différent** de l'uid Firebase Auth, l'accès direct `<collection>/{uid}` pointe vers un document inexistant.

Or les documents métier (commandes, lignes, etc.) portent souvent le **`<entite>_uid`** (uid Auth dénormalisé), pas le doc.id.

Anti-pattern : `servicePush.envoyer(collection: '<collection>', docId: uid)` → `<collection>/{uid}` n'existe pas → **0 token → aucun push**, en silence (best-effort).

## Règle

Avant tout push/lecture de tokens côté backend quand `doc.id ≠ uid`, **résoudre le doc.id** :

```dart
final snap = await firestore
    .collection('<collection>')
    .where('uid', WhereFilter.equal, entiteUid)
    .limit(1)
    .get();
if (snap.docs.isEmpty) return; // skip non bloquant
final docId = snap.docs.first.id; // → <collection>/{docId}.fcm_tokens
```

## Exceptions

- Si la collection est **clé par uid** (`<collection>/{uid}` existe), push direct sans résolution.
- Si un jour `doc.id == uid` est garanti, cette règle devient un no-op mais reste correcte (la requête renvoie le doc).
