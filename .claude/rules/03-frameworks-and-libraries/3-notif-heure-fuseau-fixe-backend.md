---
description: Côté backend (runtime UTC, ex. Dart Frog sur Cloud Run), formater une heure destinée à l'utilisateur (notifications push/email) avec un décalage régional EXPLICITE — jamais `.toLocal()`, qui renvoie l'UTC sur un serveur UTC (heures décalées). Valable pour une région à offset fixe (sans heure d'été).
globs: apps/*_api/**/*.dart
alwaysApply: false
---
- **Contexte** : le runtime serveur (ex. Cloud Run) est en **UTC**. `Timestamp.toDate()` / `.toLocal()` y renvoient l'heure UTC, pas l'heure locale régionale. Bug typique : un événement à 13h00 annoncé « 09h00 » dans la notif (offset oublié).
- **Anti-pattern** :
  ```dart
  final dt = DateTime.fromMillisecondsSinceEpoch(ts.seconds * 1000, isUtc: true)
      .toLocal(); // ⚠ = UTC sur un serveur UTC, PAS l'heure régionale
  final heure = '${dt.hour}h${dt.minute}';
  ```
- **Correct** — décalage régional fixe (région sans DST ; remplacer `4` par l'offset cible) :
  ```dart
  // helper : apps/<app>_api/lib/notifications/heure_region.dart
  final dt = DateTime.fromMillisecondsSinceEpoch(ts.seconds * 1000, isUtc: true)
      .add(const Duration(hours: 4)); // région = UTC+4 toute l'année
  ```
- **À NE PAS confondre** avec `3-firestore-datetime-utc.mdc` : celle-ci traite le round-trip de modèle + l'affichage côté **CLIENT** (device dans la région → `.toLocal()` OK). Ici on est côté **BACKEND** (serveur UTC → `.toLocal()` ≠ région).
- **Portée** : toute heure affichée à l'utilisateur produite par le backend (corps de notifications push / email).
- **Limite** : valable uniquement pour une région à offset UTC fixe. Pour une région avec heure d'été, utiliser une vraie librairie de fuseaux (`timezone`).
