---
description: Tout getter front qui combine une `DateTime` saisie + un `TimeOfDay` doit retourner un `DateTime.utc()` représentant l'heure murale (wall time) de la région cible, indépendamment du fuseau du navigateur. Sinon, un opérateur distant (ex. navigateur en CEST UTC+2) envoie un instant décalé → le backend ré-ajuste → décalage cumulé sur le créneau métier.
globs: apps/<app>/lib/**/*state.dart, apps/<app>/lib/**/*bloc.dart, apps/<app>/lib/**/*saisie*.dart
alwaysApply: false
---

# Saisie d'un créneau = wall time d'un fuseau à décalage fixe

## Symptôme
Un opérateur saisit `19/05/2026 18:55` dans un formulaire (nouvelle entrée, modification, contre-proposition). Sur une machine dont le navigateur est dans un autre fuseau (ex. UTC+2), le backend reçoit un créneau décalé → tarif faux, logique multi-tranches mal déclenchée, document persisté avec une mauvaise heure.

## Cause
Le getter typique combinant date + heure utilise `DateTime(year, month, day, hour, minute)` qui produit un `DateTime` LOCAL navigateur. `.toUtc()` côté sérialisation HTTP fait `local→UTC` selon la TZ du navigateur → l'opérateur distant envoie un instant différent de celui qu'il a saisi. Le backend (qui applique un offset fixe pour reconstruire l'heure régionale) reconstruit alors une heure fausse.

## Règle

1. **Getter `creneauX: DateTime`** dans bloc/state (remplacer `4` par l'offset fixe de la région cible) :
   ```dart
   DateTime? get creneauDepart {
     final date = dateDepartSaisie;
     final heure = heureDepartSaisie;
     if (date == null || heure == null) return null;
     // L'opérateur saisit toujours en heure murale de la région cible
     // (fuseau à offset fixe, sans heure d'été). On retourne l'instant
     // UTC correspondant, indépendant du navigateur.
     return DateTime.utc(
       date.year, date.month, date.day, heure.hour, heure.minute,
     ).subtract(const Duration(hours: 4));
   }
   ```

2. **Formatage côté affichage** : reconvertir en wall time régional pour montrer l'heure saisie :
   ```dart
   static String _formaterCreneau(DateTime? creneau) {
     if (creneau == null) return '—';
     final wallTime = creneau.add(const Duration(hours: 4));
     return DateFormat('EEEE d MMMM à HH:mm', 'fr_FR').format(wallTime);
   }
   ```

3. **Hors-scope** : si la `DateTime` provient déjà d'une lecture (Firestore read), pas besoin de reconvertir — Firestore retourne déjà UTC via `Timestamp.toDate().toUtc()` (cf. règle `3-firestore-datetime-utc.mdc`). Seul le **point de saisie** doit appliquer cette convention.

## Points d'application
- Tout formulaire qui saisit un créneau métier (création, modification, contre-proposition).
- Tout nouvel écran de saisie d'horaire devant être interprété dans le fuseau régional, pas celui du navigateur.

## Note
Valable uniquement pour une région **à décalage UTC fixe** (sans heure d'été) : l'offset constant peut être codé en dur. Pour une région avec DST, passer par une vraie librairie de fuseaux (`timezone`).
