---
description: Tout shell racine multi-pages (web) doit basculer sur `Scaffold(drawer:)` + bouton hamburger en mode mobile (< 768 px) via `LayoutBuilder` + un helper de breakpoint. Sinon, une sidebar fixe (ex. 240 px) écrase le body sur écran mobile (grande partie de la largeur mangée, cards/KPI en overflow).
globs: apps/<app>/lib/**/views/**.dart
alwaysApply: false
---

# Shell responsive — Drawer mobile, sidebar permanente tablet/desktop

## Quand
Toute vue racine qui assemble un menu latéral (sidebar fixe) + un body dynamique piloté par un bloc de navigation. Cas typique : shell d'accueil, shells de modules déportés.

## Règle

Wrapper le `Scaffold` dans un `LayoutBuilder` qui résout le breakpoint via un helper centralisé (ex. `core/responsive/breakpoints.dart`) :

```dart
LayoutBuilder(
  builder: (context, constraints) {
    final breakpoint = breakpointDepuisLargeur(constraints.maxWidth);

    if (breakpoint.estMobile) {
      return Scaffold(
        drawer: const SizedBox(
          width: kLargeurMenuLateral,
          child: Drawer(child: MenuLateral()),
        ),
        body: SafeArea(
          child: Column(
            children: <Widget>[
              // Bouton hamburger (titre absent — sous-pages ont leur AppBar).
              Builder(
                builder: (innerContext) => SizedBox(
                  height: 48,
                  child: Align(
                    alignment: Alignment.centerLeft,
                    child: IconButton(
                      tooltip: localisation.shellOuvrirMenuTooltip,
                      icon: const Icon(Icons.menu),
                      onPressed: () => Scaffold.of(innerContext).openDrawer(),
                    ),
                  ),
                ),
              ),
              const Expanded(child: _FlowBuilderShell()),
            ],
          ),
        ),
      );
    }

    // Tablet + desktop : sidebar permanente.
    return const Scaffold(
      body: SafeArea(
        child: Row(
          children: <Widget>[
            SizedBox(width: kLargeurMenuLateral, child: MenuLateral()),
            VerticalDivider(width: 1),
            Expanded(child: _FlowBuilderShell()),
          ],
        ),
      ),
    );
  },
)
```

## Notes

- Le `Builder` est OBLIGATOIRE pour que `Scaffold.of(innerContext).openDrawer()` trouve le `Scaffold` parent (sinon `Scaffold.of` remonte plus haut et plante).
- Pas de titre dans la barre top mobile : les sous-pages ont leur propre AppBar avec leur titre. Le hamburger global est SEPARÉ — accepter la double bande sur mobile.
- `MenuLateral` peut rester inchangé : il sait se rendre en Container plein de hauteur, compatible Drawer.
- Breakpoint `< 768 px` couvre tous les téléphones modernes. Tablet (≥ 768 px) garde la sidebar permanente.
