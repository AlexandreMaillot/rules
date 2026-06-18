---
description: PLACER les helpers de routes Dart Frog HORS du dossier `routes/` LORS de l'extraction d'un fichier helper depuis un handler.
globs: apps/*_api/routes/**/*.dart, apps/*_api/lib/**/*.dart
alwaysApply: true
---

Principe :
- Dart Frog **scanne tous les fichiers `.dart`** du dossier `routes/` et les enregistre comme handlers HTTP — indépendamment du préfixe (`_`, `private_`, etc.).
- Le préfixe `_` (privé Dart compile-time) ne suffit PAS : Dart Frog ignore la convention de visibilité Dart et tente toujours d'appeler `<fichier>.onRequest(context)` → erreur build `Method not found: 'onRequest'`.

Pattern obligatoire — helper de route :
- Placer dans `apps/<app>_api/lib/<feature>/` avec préfixe explicite `route_*` :
  ```
  apps/<app>_api/
  ├── lib/
  │   └── facturation/
  │       ├── service_export_csv.dart
  │       └── route_export_csv_helpers.dart   ← helper, hors routes/
  └── routes/
      └── facturation/
          ├── _middleware.dart                 ← OK (convention Dart Frog reconnue)
          └── export-csv.dart                  ← handler, importe via package:
  ```
- Importer dans le handler :
  ```dart
  import 'package:<app>_api/facturation/route_export_csv_helpers.dart';
  ```

Conventions Dart Frog reconnues (autorisées dans `routes/`) :
- `<route>.dart` → handler avec `onRequest(context)`
- `_middleware.dart` → middleware avec `middleware(handler)` (convention Dart Frog)
- Sous-dossiers → routes nested

À éviter dans `routes/` :
- ❌ `_helpers.dart` ou `_<nom>_helpers.dart` (Dart Frog tente d'enregistrer comme route)
- ❌ `private_*.dart`
- ❌ `__internal_*.dart`
- ❌ Toute extension `.dart` qui n'est pas un handler ou `_middleware.dart`

Bug typique :
- `routes/facturation/_export_csv_helpers.dart` (extrait d'un handler lors d'un refacto) casse le build :
  ```
  .dart_frog/server.dart: Error: Method not found: 'onRequest'.
      ..all('/_export_csv_helpers', (context) => facturation_export_csv_helpers.onRequest(context,))
  ```
- Fix : déplacer en `lib/facturation/route_export_csv_helpers.dart` + import via `package:`.
