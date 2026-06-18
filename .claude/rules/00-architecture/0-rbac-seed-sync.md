---
description: APPLIQUER la synchronisation entre la définition Dart des droits/rôles et le script de seed serveur LORS de l'ajout d'un nouveau droit ou d'un nouveau rôle dans le système de permissions (RBAC).
globs: apps/<app>/lib/**/permissions/*.dart, scripts/seed_roles.mjs
alwaysApply: true
---

Principe :
- Le **middleware serveur vérifie les droits côté Firestore** (`_roles/{roleId}.droits`), pas côté Dart compile-time.
- La **source de vérité Firestore (prod)** est un script de seed (`scripts/seed_roles.mjs`, firebase-admin SDK qui bypass `allow write: if false`).
- Toute divergence entre la liste Dart et la liste JS = bug runtime 403 (typiquement vécu lors de l'ajout d'un droit côté Dart sans miroir côté seed).

Checklist obligatoire à l'ajout d'un nouveau droit :
1. [ ] `apps/<app>/lib/**/permissions/droits.dart` — ajout de la définition du droit (`cle`, `nom`, `description`) dans la liste des droits applicatifs.
2. [ ] `apps/<app>/lib/**/permissions/roles_seed.dart` — ajout dans `kRoleAdmin.droits` (et les autres rôles si pertinent).
3. [ ] `scripts/seed_roles.mjs` — ajout dans `ADMIN_DROITS` et/ou les autres listes de rôles (miroir exact).
4. [ ] Run `node scripts/seed_roles.mjs --project=<projectId>` après merge.
5. [ ] Tests `droits_test.dart` + `roles_seed_test.dart` mis à jour.

Vérification automatique :
- Un **test de drift** (`seed_drift_test.dart`) parse le script JS et le compare avec la liste Dart. **Échec rouge si drift**. C'est la seule mitigation robuste — la discipline manuelle échoue.

Cas particulier — nouveau rôle :
- Ajouter dans `roles_seed.dart` : nouvelle constante `kRole<Nom>`.
- Ajouter dans `seed_roles.mjs` : nouveau `seedRole('<id>', '<libellé>', DROITS)`.
- Vérifier côté backend : si le `role_id` d'un utilisateur lit un doc de rôle inexistant → `aDroit() = false` → 403.

Pourquoi :
- Le bug n'est **pas détectable** à `flutter analyze` ni à `dart test` côté Dart — il dépend de l'état Firestore.
- Le test de drift en CI ferme la fenêtre.
