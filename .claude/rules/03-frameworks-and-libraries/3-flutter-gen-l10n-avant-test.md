---
description: "`flutter gen-l10n` est obligatoire avant `flutter test` après tout merge qui modifie un fichier ARB (`app_fr.arb`/`app_en.arb`), car le dossier `lib/l10n/gen/` est gitignored — les fichiers générés ne sont pas commités."
globs:
  - "apps/*/lib/l10n/arb/app_*.arb"
alwaysApply: false
---

# Flutter — `flutter gen-l10n` avant `flutter test` post-changement ARB

## Pourquoi

Les fichiers générés par `flutter gen-l10n` (`AppLocalizations`, getters typés par clé i18n) vivent dans `apps/<app>/lib/l10n/gen/` qui est **gitignored**.

Conséquences :
- Le fichier `app_localizations.dart` n'est pas commité.
- CI le régénère via `flutter pub get` + `flutter gen-l10n` (auto).
- **Localement, après un merge** qui change les ARB (ex. 2 PRs ajoutant chacune ~25 clés), le fichier généré local est **obsolète**.
- `flutter test` échoue avec « The getter 'maNouvelleCle' isn't defined for the type 'AppLocalizations' » → erreurs de compilation transitive sur de nombreux fichiers.

## Règle

**Après tout merge qui modifie `app_fr.arb` ou `app_en.arb`** (ou les 2), avant de lancer les tests localement :

```bash
cd apps/<app>
flutter gen-l10n
flutter test
```

**Dans tout brief d'agent** qui doit tester localement après modif ARB, inclure :

> Tests : `cd apps/<app> && flutter gen-l10n && flutter test`
> Le `flutter gen-l10n` est requis dans un worktree neuf (le dossier `lib/l10n/gen/` est gitignored).

## Anti-pattern

- ❌ Lancer `flutter test` directement après un `git checkout` ou `git worktree add` neuf → conclure à tort à une régression.
- ❌ Pousser un commit de panique qui ajoute le fichier généré au repo → casser le `.gitignore` et créer une dette de drift CI vs local.

## Note CI

Le CI ne souffre pas de ce piège car `flutter pub get` + `flutter gen-l10n` sont dans le workflow de tests. Seuls les agents et le développement local sont concernés.
