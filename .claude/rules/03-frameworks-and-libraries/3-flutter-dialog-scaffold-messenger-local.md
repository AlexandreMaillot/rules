---
description: "`ScaffoldMessenger.maybeOf(context)` invoqué depuis un `Dialog` (poussé par `showDialog`) remonte au Scaffold parent DERRIÈRE le Dialog → `SnackBar` masqués. Fix : envelopper le contenu du Dialog dans `ScaffoldMessenger(Scaffold(backgroundColor: transparent, body: ...))` pour avoir un scope local."
globs:
  - "apps/*/lib/**/views/*.dart"
alwaysApply: false
---

# Flutter — `ScaffoldMessenger` local obligatoire dans un `Dialog`

## Symptôme

Tu pousses un Dialog via `showDialog<T>(context, builder: ...)`. Dans le contenu de la Dialog, un `BlocListener` (ou similaire) dispatch :
```dart
ScaffoldMessenger.maybeOf(context).showSnackBar(SnackBar(...));
```
Côté UX : **le SnackBar est invisible**. Aucune erreur Flutter, juste masqué derrière la modal Dialog.

Cause : `ScaffoldMessenger.maybeOf(context)` remonte l'arbre de widgets et trouve le `ScaffoldMessenger` du `Scaffold` parent — qui est **DERRIÈRE le Dialog**, donc occulté par la barrier modale + le contenu du Dialog.

## Règle

Tout `Dialog` (ou modal route équivalent) qui doit afficher un `SnackBar` depuis son contenu DOIT envelopper son `child:` dans un `ScaffoldMessenger` + `Scaffold` locaux :

```dart
Dialog(
  backgroundColor: colorScheme.surface,
  child: ConstrainedBox(
    constraints: const BoxConstraints(maxWidth: 343),
    // Scope local pour que ScaffoldMessenger.maybeOf(context) trouve un
    // messager AU-DESSUS de la barrier modale (pas derrière).
    child: ScaffoldMessenger(
      child: Scaffold(
        backgroundColor: Colors.transparent,
        body: <ton contenu réel ici>,
      ),
    ),
  ),
)
```

Le `Scaffold` interne est obligatoire (le `ScaffoldMessenger` cherche un `Scaffold` descendant pour ancrer ses SnackBar). `backgroundColor: Colors.transparent` préserve l'apparence du Dialog parent.

## Anti-patterns

- ❌ `ScaffoldMessenger.of(navigatorContext.context)` ou `findRootAncestor` pour récupérer le messager parent : techniquement fonctionne mais le SnackBar apparaît **derrière** la modal (masqué par la barrier).
- ❌ Afficher les erreurs uniquement en inline (sans SnackBar) — perd l'UX cohérente avec le reste de l'app.
- ❌ Pop la modal AVANT d'afficher le SnackBar : marche pour les erreurs terminales mais pas pour les erreurs transitoires (rate limit, réseau) où la modal doit rester ouverte.

## Cas spécial : pop modal + SnackBar persistant après

Si l'erreur doit pop la modal puis afficher un SnackBar persistant sur l'écran parent, récupérer le `ScaffoldMessenger` parent AVANT le pop :

```dart
final messager = ScaffoldMessenger.maybeOf(
  Navigator.of(context, rootNavigator: true).context,
);
Navigator.of(context).pop();
messager?.showSnackBar(SnackBar(
  content: Text('Erreur persistante...'),
  duration: const Duration(seconds: 8),
));
```

## Référence

- Flutter docs : `ScaffoldMessenger.maybeOf` recherche l'ancêtre, pas le descendant.
