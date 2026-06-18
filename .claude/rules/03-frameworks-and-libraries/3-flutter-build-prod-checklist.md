---
description: Build prod Flutter (Android + iOS) — checklist pour éviter les pièges silencieux (key.properties orphelin, iOS deployment target désynchronisé entre 3 fichiers, pod install sans repo-update après bump Firebase).
globs:
  - "apps/*/android/app/build.gradle.kts"
  - "apps/*/android/key.properties"
  - "apps/*/ios/Podfile"
  - "apps/*/ios/Runner.xcodeproj/project.pbxproj"
alwaysApply: false
---

# Flutter — checklist build prod (signing Android + deployment target iOS)

## Pourquoi

Pièges typiques au premier build store :

- `flutter build apk --flavor production --release` →
  `SigningConfig "release" is missing required property "storeFile"`.
  Cause : `key.properties` placé à la racine de l'app au lieu de
  `android/`. `app/build.gradle.kts` lit `rootProject.file("key.properties")`
  qui résout vers `android/key.properties`. Fichier gitignored donc piège
  invisible à `git diff` / revue PR.
- `flutter build ios --flavor production --release` →
  `cloud_firestore requires higher minimum iOS deployment version`.
  Cause : ligne `platform :ios, '13.0'` commentée dans le `Podfile` →
  CocoaPods retombe sur un iOS par défaut trop bas alors que Firebase
  récent exige iOS plus récent. Et `project.pbxproj` a plusieurs
  occurrences de `IPHONEOS_DEPLOYMENT_TARGET` à aligner.

## Règles

### Android — signing config

1. `apps/<slug>/android/key.properties` (PAS à la racine de l'app)
   avec :
   ```
   storePassword=...
   keyPassword=...
   keyAlias=<slug>_keystore
   storeFile=../<slug>_keystore.jks
   ```
   `storeFile` est relatif au sous-projet `:app` (donc `android/app/`),
   d'où le `../` pour remonter à `android/<slug>_keystore.jks`.
2. Tout fichier `key.properties` ailleurs (ex. racine de l'app) est
   orphelin — Gradle ne le lit pas. À supprimer.
3. Le keystore vit à `apps/<slug>/android/<slug>_keystore.jks`
   (gitignored via `**/*.jks` dans `android/.gitignore`).

### iOS — deployment target (3 sources à aligner)

Quand on bump Firebase SDK (ou tout pod qui exige un iOS plus récent),
mettre à jour les **3** fichiers ensemble :

1. **`ios/Podfile`** :
   ```ruby
   platform :ios, '15.6'  # décommenté + valeur cible

   post_install do |installer|
     installer.pods_project.targets.each do |target|
       flutter_additional_ios_build_settings(target)
       target.build_configurations.each do |config|
         config.build_settings['IPHONEOS_DEPLOYMENT_TARGET'] = '15.6'
       end
     end
   end
   ```
   Le `post_install` est obligatoire : sans lui, certains pods
   compilent à leur deployment target par défaut et provoquent des
   warnings au mieux, des erreurs de link au pire.

2. **`ios/Runner.xcodeproj/project.pbxproj`** :
   remplacer **toutes** les occurrences de `IPHONEOS_DEPLOYMENT_TARGET`
   (typiquement 3 build configs × N flavors).

3. **`ios/Flutter/AppFrameworkInfo.plist`** : si la clé
   `MinimumOSVersion` est présente, l'aligner. Sur les versions récentes
   de Flutter, CocoaPods la déduit du Podfile et la supprime à
   `pod install` — c'est OK, committer la suppression.

### Réinstall CocoaPods après bump Firebase

```bash
cd apps/<slug>/ios
rm -rf Pods Podfile.lock .symlinks
cd .. && flutter pub get
cd ios && pod install --repo-update
```

Le `--repo-update` est obligatoire si la spec locale ne contient pas
encore la version Firebase demandée (sinon échec
`None of your spec sources contain a spec satisfying...`).

## Anti-patterns

- **Copier un Podfile d'exemple sans filtrer** :
  `pod 'Firebase/Messaging'`, `pod 'GoogleUtilities'`, certaines
  `GCC_PREPROCESSOR_DEFINITIONS`, `SWIFT_OPTIMIZATION_LEVEL = '-Onone'`
  ne sont **pas** universels. Toujours croiser avec `pubspec.yaml` :
  pas de `firebase_messaging` ni `permission_handler` → ne PAS ajouter
  ces lignes.
- **Toucher au Podfile sans relancer `pod install`** : le workspace
  embarque encore l'ancien état. Toujours réinstaller.
- **Bumper Firebase dans `pubspec.yaml` sans bumper `platform :ios`** :
  le build échoue tard (résolution Pods), alors qu'un check du
  changelog Firebase aurait évité l'aller-retour.

## Side-effects à committer

Après un `pod install` propre, ces fichiers peuvent légitimement
changer et doivent être committés :

- `ios/Podfile.lock`
- `ios/Flutter/AppFrameworkInfo.plist` (perte de `MinimumOSVersion`)
- `ios/Runner.xcodeproj/xcshareddata/xcschemes/Runner.xcscheme`
- `ios/Runner.xcworkspace/contents.xcworkspacedata`
- `.gitignore` (ajouts CocoaPods générés)

Ne pas reverter par réflexe : ce sont des artefacts de l'intégration.
