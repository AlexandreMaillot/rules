---
description: Border non-uniforme + borderRadius interdit en Flutter — utiliser un overlay `Positioned` + `IgnorePointer` pour une bordure d'accentuation colorée.
globs:
  - "**/*.dart"
alwaysApply: false
---

# Flutter — Border non-uniforme + borderRadius interdit

## Pourquoi

Flutter rejette à la peinture toute `BoxDecoration` qui combine :
- un `Border` avec des `BorderSide` de couleur DIFFÉRENTE selon les côtés (ex.
  bordure gauche colorée + 3 autres côtés gris)
- ET un `borderRadius` non-zéro

Erreur lors du test/run :
```
A borderRadius can only be given on borders with uniform colors.
```

## Règle

Pour une card avec **bordure d'accentuation** (gauche colorée 4px) +
coins arrondis, utiliser un overlay `Positioned` + `IgnorePointer` :

```dart
final card = Material(
  color: colorScheme.surface,
  borderRadius: BorderRadius.circular(Rayons.l),
  child: Container(
    decoration: BoxDecoration(
      border: Border.all(color: colorScheme.outlineVariant), // uniforme
      borderRadius: BorderRadius.circular(Rayons.l),
    ),
    child: ...,
  ),
);

if (couleurAccent == null) return card;

return ClipRRect(
  borderRadius: BorderRadius.circular(Rayons.l),
  child: Stack(
    children: [
      card,
      Positioned(
        left: 0,
        top: 0,
        bottom: 0,
        child: IgnorePointer(
          child: Container(width: 4, color: couleurAccent),
        ),
      ),
    ],
  ),
);
```

Le `IgnorePointer` est CRUCIAL pour que les taps (InkWell) passent à
travers la bande vers la card. Le `ClipRRect` autour du Stack sert à
masquer la bande aux 2 coins arrondis.

## Bypass acceptables

- Bordure uniforme partout → `Border.all(color: X)` + `borderRadius`
  fonctionne nativement.
- Pas de coins arrondis → `Border(left: BorderSide(...), top: ...)`
  hétérogène fonctionne SANS `borderRadius`.
- Cas extrême : `ShapeDecoration` avec un `OutlinedBorder` custom
  (verbose, à éviter sauf si vraiment nécessaire).
