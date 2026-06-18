---
description: APPLIQUER un nommage FR 100% pour le code Dart LORS de l'écriture de nouveau code ou de plans côté frontend (projets dont la convention est le français).
globs: apps/*/lib/**/*.dart, apps/*/test/**/*.dart, aidd_docs/tasks/**/*.md
alwaysApply: true
---

Principe:
- Tout le code Dart côté apps utilise des noms **français** : classes, events, states, enums, variables, fichiers, dossiers.
- La BDD Firestore utilise des **collections en français snake_case** (ex: `demandes`, `paiements`, `alertes`) pour cohérence FR 100% du projet. Les **champs** à l'intérieur des documents restent en snake_case (ex: `heure_creation`, `nom_client`). Mapping snake_case → camelCase dans `core/modeles/*.fromFirestore()` via `String.enCamelCase` + `Enum.values.byName()`.
- Les termes techniques transverses qui n'ont pas d'équivalent français courant peuvent rester en anglais (`Bloc`, `Event`, `State`, `Widget`, `Repository`) — uniquement comme suffixes.

Conventions de nommage:
- **Classes** : `ConnexionPage`, `TableauDeBordView`, `NavigationBloc`, `DroitsEtat`
- **Events** (suffixe en français) : `ConnexionEmailChange`, `AppUtilisateurChange`, `NavigationPageDemandee`
- **States** (suffixe `Etat`) : `AppEtat`, `ConnexionPageEtat`, `NavigationEtat`
- **Enums** : `StatutAuth { inconnu, deconnecte, connecte }`, `PageAdmin { tableauDeBord, ... }`, `RoleUtilisateur { admin, inconnu }`
- **Erreurs** (sealed) : `ErreurConnexion`, `ErreurResetMdp`, `ErreurEmail`
- **Repositories / "Repository"** : `DepotAuth`, `DepotVehicules` (préfixe `Depot` au lieu de suffixe `Repository`)
- **Fichiers** : `snake_case` FR — `connexion_page_bloc.dart`, `role_utilisateur.dart`, `depot_auth.dart`
- **Dossiers** : `pages/connexion/`, `core/navigation/`, `pages/accueil/widgets/`
- **Clés i18n** : camelCase FR — `connexionTitre`, `sidebarDeconnexion`, `erreurConnexionIdentifiantsInvalides`

Mapping technique → FR (référence):
- `Login` → `Connexion`
- `Logout` → `Deconnexion`
- `Password` → `MotDePasse` ou `Mdp` (abréviation tolérée dans les noms de fichiers/classes)
- `User` → `Utilisateur`
- `Settings` → `Parametres`
- `Dashboard` → `TableauDeBord`
- `Shell` → `Accueil` ou `Structure` (plus générique)
- `Loading` → `Chargement`
- `Drawer/Sidebar` → `MenuLateral`

Exceptions explicites:
- Package Flutter : `MaterialPage`, `Scaffold`, `TextField`, etc. — inchangés
- Classes officielles Firebase : `FirebaseAuth`, `User`, `FirebaseAuthException` — inchangées (mappées vers nos sealed classes FR `ErreurConnexion` dans le depot)
- Suffixes techniques : `Bloc`, `Event`, `State`

Plans (fichiers `aidd_docs/tasks/**/*.md`):
- Les noms de classes/events/states proposés doivent être en français.
- Si un plan est généré depuis une maquette avec des labels EN, traduire avant de les nommer côté Dart.

Interdictions:
- `class LoginPage`, `class UserBloc`, `class DashboardState`, `enum UserRole` — tous en anglais
- `login_page.dart`, `user_bloc.dart`, `dashboard_state.dart` — tous en anglais
- Variables anglaises dans le corps (`final user = ...`, préférer `final utilisateur = ...`)
