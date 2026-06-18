---
description: Toute fonction qui itère sur des frontières temporelles (`isAfter`/`isBefore` sur DateTime) doit garder un flag UTC cohérent entre le curseur et les frontières construites. Sinon, sur un serveur/test runner hors UTC, les comparaisons `millisecondsSinceEpoch` mixent LOCAL et UTC et certaines frontières sont silencieusement ratées.
globs: apps/*_api/lib/**/*.dart, apps/**/lib/**/*decoupeur*.dart, apps/**/lib/**/*frontiere*.dart
alwaysApply: false
---

# Cohérence flag UTC dans le découpage temporel

## Symptôme
Fonction qui découpe une période en sous-périodes selon des frontières temporelles (07h, 19h, minuit, etc.). Tests verts en local, comportement faux en prod (selon TZ du serveur vs runner CI vs machine dev). Une frontière attendue n'est pas créée → 1 tranche au lieu de 2, ou résultat tarifaire faux.

## Cause
Mix `DateTime` LOCAL (TZ système) et `DateTime` UTC marqué dans la même boucle :
- Le curseur (entrée) peut être marqué UTC (`DateTime.utc(...)` ou sortie d'un ajustement de fuseau).
- Les frontières internes sont construites avec `DateTime(y, m, d, h)` (LOCAL système).
- `a.isAfter(b)` / `a.isBefore(b)` comparent `millisecondsSinceEpoch` qui dépend du flag.
- Selon la TZ machine, la comparaison est correcte par accident OU mauvaise silencieusement.

## Règle

1. **Forcer le flag UTC du curseur à l'entrée** :
   ```dart
   final ajuste = ajusterFuseau(creneauDepart);
   final tDepart = ajuste.isUtc
       ? ajuste
       : DateTime.utc(
           ajuste.year, ajuste.month, ajuste.day,
           ajuste.hour, ajuste.minute, ajuste.second, ajuste.millisecond,
         );
   ```

2. **Construire toutes les frontières internes en `DateTime.utc(...)`** :
   ```dart
   // ❌ Mauvais — DateTime LOCAL
   final minuit = DateTime(t.year, t.month, t.day + 1);

   // ✅ Bon — DateTime UTC marqué, cohérent avec t
   final minuit = DateTime.utc(t.year, t.month, t.day + 1);
   ```

3. **Si un sous-helper applique son propre ajustement TZ**, passer un `DateTime` *naïf* (non-UTC, composants déjà ajustés) pour éviter le double-ajustement :
   ```dart
   DateTime _enWallTime(DateTime ajuste) {
     return DateTime(  // non-UTC
       ajuste.year, ajuste.month, ajuste.day,
       ajuste.hour, ajuste.minute, ajuste.second, ajuste.millisecond,
     );
   }
   ```

## Test régression obligatoire

Pour tout module de découpage temporel, ajouter UN test avec **`DateTime.utc(...)`** en entrée (pas `DateTime(...)` LOCAL). Les tests `DateTime LOCAL` cohérents avec une frontière `LOCAL` masquent le bug :

```dart
test('régression bug double-ajustement timezone (entrée UTC)', () {
  final tranches = decouperEnTranchesTarifaires(
    creneauDepart: DateTime.utc(2026, 5, 19, 14, 55),
    duree: const Duration(minutes: 24),
    typeTrajet: TypeTrajet.allerSimple,
  );
  expect(tranches.length, 2);
});
```
