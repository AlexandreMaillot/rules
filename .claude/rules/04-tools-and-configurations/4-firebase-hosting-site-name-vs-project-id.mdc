---
description: Le nom du site Firebase Hosting (déclaré `"site"` dans `firebase.json`) est DIFFÉRENT de l'ID projet GCP. Confondre les deux dans un script de deploy provoque `Error: Hosting site or target X not detected in firebase.json` au moment du `firebase deploy --only hosting:X`.
globs: scripts/**/*.sh, apps/**/firebase.json
alwaysApply: false
---

# Firebase Hosting site name ≠ project ID

## Symptôme

```
==> Firebase Hosting deploy
Error: Hosting site or target <project-id> not detected in firebase.json
```

Le `firebase deploy --only hosting:<site>` échoue parce que le nom passé en argument ne match aucun `"site"` ou `"target"` déclaré dans `firebase.json`.

## Cause

Confusion classique entre :
- **ID projet GCP** : ex. `mon-projet-dev` (utilisé partout : `gcloud config set project`, secrets, etc.).
- **Nom site Firebase Hosting** : ex. `mon-admin-dev` (= sous-domaine `mon-admin-dev.web.app`, déclaré `"site"` dans `firebase.json`).

Un site Hosting est nommé séparément du projet (un projet peut héberger plusieurs sites).

## Règle

1. **Avant de hardcoder un site Hosting dans un script de deploy**, vérifier le bon nom via :
   ```bash
   grep -A1 '"site"' firebase.json | head -3
   # ou
   firebase hosting:sites:list --project=<PROJECT_ID>
   ```

2. **Ne JAMAIS supposer** que `FIREBASE_HOSTING_SITE = PROJECT_ID`. Les nommer explicitement et séparément dans toute config de deploy :

   ```bash
   # ❌ Mauvais — réutilise PROJECT_ID
   FIREBASE_HOSTING_SITE="${PROJECT_ID}"

   # ✅ Bon — nom site dédié, vérifié dans firebase.json
   PROJECT_ID="mon-projet-dev"
   FIREBASE_HOSTING_SITE="mon-admin-dev"
   ```

3. **Multi-environnements** : déclarer un mapping explicite `env → site` :
   ```bash
   case "${env}" in
     dev)     FIREBASE_HOSTING_SITE="mon-admin-dev" ;;
     staging) FIREBASE_HOSTING_SITE="mon-admin-staging" ;;
     prod)    FIREBASE_HOSTING_SITE="mon-admin" ;;
   esac
   ```

## Référence

- `firebase.json` racine : champ `"hosting.site"`.
