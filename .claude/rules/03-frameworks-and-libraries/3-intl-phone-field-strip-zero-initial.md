---
description: "`IntlPhoneField` (package `intl_phone_field`) concatène naïvement `countryCode` + `number` sans normaliser. Un utilisateur qui saisit un numéro avec `0` initial (format national) alors qu'un indicatif international est sélectionné produit un numéro invalide → Firebase Auth rejette. Fix : strip le `0` initial dans `onChanged` AVANT d'envoyer à Firebase, sans toucher au champ visible."
globs:
  - "apps/*/lib/**/*phone*.dart"
  - "apps/*/lib/**/*telephone*.dart"
alwaysApply: false
---

# IntlPhoneField — strip `0` initial pour saisie format national avec indicatif sélectionné

## Symptôme

Le placeholder affiche un format local habituel (ex. `Ex. 06 92 12 34 56`) pour guider l'UX. L'indicatif pays (ex. `+262`) est déjà sélectionné via le drapeau `IntlPhoneField`.

Si l'utilisateur saisit `06 92 12 34 56` (par habitude, format national), `IntlPhoneField` retourne :
- `phone.number = '0692123456'`
- `phone.completeNumber = '+2620692123456'`  ← **invalide** (indicatif + `0` parasite)

Firebase Auth rejette avec `firebase_auth/invalid-phone-number` (ou silencieusement selon les versions).

## Règle

Dans `onChanged` du `IntlPhoneField`, strip le `0` initial sur le **numéro envoyé en aval** (Firebase Auth / bloc) SANS toucher au champ visible :

```dart
IntlPhoneField(
  ...
  onChanged: (phone) {
    // L'utilisateur saisit « 06 92 … » par habitude (format national)
    // alors que l'indicatif est déjà sélectionné — IntlPhoneField concatène
    // alors `indicatif` + `0692…` → numéro invalide. On strip le `0` initial
    // dans la valeur envoyée à Firebase Auth SANS toucher au champ visible.
    final numeroBrut = phone.number;
    final numeroSansZeroInitial = numeroBrut.startsWith('0')
        ? numeroBrut.substring(1)
        : numeroBrut;
    final numeroE164Corrige = '${phone.countryCode}$numeroSansZeroInitial';

    context.read<MonBloc>().add(ChangerNumero(
      numero: numeroBrut,            // ← brut pour validation longueur
      numeroE164: numeroE164Corrige, // ← corrigé pour Firebase Auth
      numeroValide: ...,
    ));
  },
)
```

**Pourquoi ne pas modifier le champ visible** : garder visuellement le `0` car sinon les utilisateurs non habitués au format international ne comprennent pas. Le placeholder format national reste pertinent UX. Le strip est silencieux côté backend.

## Variantes de pays

Le strip `0` initial est spécifique aux pays où le `0` est le préfixe national (FR, beaucoup de pays européens). Pour un projet multi-pays, vérifier que `phone.countryCode` correspond à un pays où cette règle s'applique :

```dart
const stripLeadingZeroCountries = {'+33', '+262', '+44', '+49', '+39', '+31', '+32'};
final shouldStrip = stripLeadingZeroCountries.contains(phone.countryCode);
final numeroFinal = (shouldStrip && phone.number.startsWith('0'))
    ? phone.number.substring(1)
    : phone.number;
```

Pour un projet mono-pays, un simple `startsWith('0')` suffit.

## Anti-patterns

- ❌ Changer le placeholder pour retirer le `0` → désoriente les utilisateurs non habitués au format international.
- ❌ Ajouter un `inputFormatters` qui strip le `0` à la frappe → l'utilisateur tape `0` et le voit disparaître → confusion.
- ❌ Strip côté backend uniquement → trop tard, Firebase Auth REST a déjà reçu et rejeté le numéro invalide.
- ❌ Validation regex avant submit (« le numéro ne doit pas commencer par 0 ») → message d'erreur déroutant.

## Référence

- Référence plan de numérotation FR : <https://www.arcep.fr/la-regulation/grands-dossiers-thematiques-transverses/la-numerotation/le-plan-de-numerotation-francais.html>
