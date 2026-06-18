---
description: Chaque nouveau site Firebase Hosting (`<site>.web.app` + `<site>.firebaseapp.com`) doit être ajouté à la whitelist CORS de l'API avant déploiement, sinon le preflight échoue silencieusement et le header `Authorization: Bearer` est strippé par le navigateur.
globs:
  - "apps/*_api/lib/config/cors.dart"
  - "firebase.json"
  - ".firebaserc"
alwaysApply: false
---

# CORS — whitelister toutes les URLs Firebase Hosting où le front est servi

## Pourquoi

Chaque site Firebase Hosting est servi sur 2 URLs par défaut :
- `https://<site>.web.app`
- `https://<site>.firebaseapp.com`

Si l'une (ou les deux) n'est pas dans la whitelist `originesAutoriseesParDefaut` de la config CORS de l'API, le navigateur :

1. Envoie un preflight OPTIONS
2. Le backend répond 204 SANS `Access-Control-Allow-Origin` (origine non whitelistée)
3. Le navigateur considère le preflight comme **échoué silencieusement**
4. Le navigateur envoie le POST quand même mais **strippe l'header `Authorization`**
5. Le middleware backend rejette « Jeton Bearer manquant » → 401

**Symptôme trompeur** : DevTools peut afficher un **400** (et non 401), avec un body venu d'une autre requête cachée. Le vrai problème est CORS.

## Règle

À chaque ajout/modification d'un site Firebase Hosting (`firebase.json` `hosting.site`) :

1. **Mettre à jour** la config CORS de l'API → `originesAutoriseesParDefaut` avec les 2 URLs :
   ```dart
   'https://<site>.web.app',
   'https://<site>.firebaseapp.com',
   ```
2. **Mettre à jour** le test CORS avec un test par URL ajoutée.
3. **Redéployer l'API** (sinon la nouvelle whitelist n'est pas active en runtime Cloud Run).
4. **Tester** :
   ```bash
   curl -sI -X OPTIONS '<cloud-run-url>/<endpoint>' \
     -H 'Origin: https://<site>.web.app' \
     -H 'Access-Control-Request-Method: POST' \
     -H 'Access-Control-Request-Headers: authorization,content-type'
   ```
   doit retourner 204 + `access-control-allow-origin: https://<site>.web.app`.

## Alternative (env var Cloud Run)

Pour un ajout ponctuel sans rebuild du backend :
```bash
gcloud run services update <service> \
  --update-env-vars=CORS_ORIGINS_ADDITIONAL=https://<site>.web.app,https://<site>.firebaseapp.com
```
**Non persistant** au prochain déploiement — réserver aux quick fixes prod.

## Anti-pattern

Ne jamais conclure « ça doit être un cache navigateur » avant d'avoir testé `curl -X OPTIONS` direct sur le backend avec l'origine concernée. Le navigateur peut afficher un mauvais code HTTP qui détourne du vrai problème CORS.
