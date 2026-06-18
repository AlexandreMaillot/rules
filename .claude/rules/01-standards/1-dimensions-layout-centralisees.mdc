---
description: UTILISER un objet de dimensions centralisé (`DimensionsLayout`) pour toute largeur max de conteneur — jamais de constante `_kLargeurMaxContenu` locale ni de magic number inline.
globs: apps/<app>/lib/**/*.dart
alwaysApply: true
---

# Règle : largeurs max centralisées via `DimensionsLayout`

## Contexte

Définir `const double _kLargeurMaxContenu = 1200;` localement dans chaque page produit :

- Vide massif sur viewport large (~ 550px de chaque côté sur ≥ 2K)
- Tableaux à nombreuses colonnes compressés inutilement
- Tout changement global (ex. passer à 1600) nécessite de modifier N pages

Centraliser les largeurs max dans `core/theme/dimensions_layout.dart` avec des constantes sémantiques.

## ❌ Interdit

```dart
// ❌ constante locale à la page
const double _kLargeurMaxContenu = 1200;

class MaPage extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: Align(
        alignment: Alignment.topCenter,
        child: ConstrainedBox(
          constraints: const BoxConstraints(maxWidth: _kLargeurMaxContenu), // ❌
```

```dart
// ❌ magic number inline
ConstrainedBox(
  constraints: const BoxConstraints(maxWidth: 1200), // ❌
```

## ✅ Obligatoire

```dart
import 'package:<app>/core/theme/dimensions_layout.dart';

class MaPageListe extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: Align(
        alignment: Alignment.topCenter,
        child: ConstrainedBox(
          constraints: const BoxConstraints(
            maxWidth: DimensionsLayout.largeurMaxContenu, // ✅
          ),
          ...
```

## Choisir la bonne constante (exemple de sémantique)

| Constante                  | Valeur | Quand l'utiliser                                                 |
| -------------------------- | ------ | ---------------------------------------------------------------- |
| `largeurMaxContenu`        | 1600   | **Listes**, **tableaux**, **dashboard**, **cartes KPI**          |
| `largeurMaxFormulaire`     | 1200   | **Formulaires 2 colonnes** (inputs textuels) — inputs restent lisibles (≈ 580px par colonne) |
| `largeurMaxDialogTravail`  | 1280   | **Dialogs de travail** (éditeur modal, plannings)                |

### Heuristique

- **Tableau ou KPI** → `largeurMaxContenu`.
- **Formulaire** avec colonnes d'inputs → `largeurMaxFormulaire`.
- **Dialog** qui couvre l'écran → `largeurMaxDialogTravail`.

## Si aucune constante n'est adaptée

**Ne pas** redéfinir localement. Ajouter une constante sémantique dans `DimensionsLayout` avec un commentaire expliquant le cas d'usage. Privilégier un nom métier (`largeurMaxDetailFiche`, `largeurMaxRapport`) plutôt qu'un nom de page (`largeurMaxFactures`).

## Cas Dialogs non-travail

Les dialogs de confirmation courts (suppression, quitter) ne sont pas concernés — ils ont leur propre `maxWidth` explicite (typiquement 480-560) défini localement avec un commentaire justifiant la taille.

## Détection dans un diff

```bash
grep -rn "_kLargeurMaxContenu\|maxWidth: 12[0-9][0-9]\|maxWidth: 16[0-9][0-9]" apps/<app>/lib/ \
  | grep -v "dimensions_layout.dart"
```

Si des lignes remontent : refacto requis vers `DimensionsLayout`.
