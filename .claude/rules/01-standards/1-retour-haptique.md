# Retour haptique à plusieurs niveaux

## Principe

- Sur mobile, chaque geste qui compte reçoit un **retour haptique** qui confirme l'action et rend l'interface plus vivante.
- Le retour est **gradué** : plus l'action est importante, plus la vibration est marquée. Une même intention a toujours le même niveau, partout dans l'app.
- Une vibration accompagne un retour visuel. Elle n'est jamais le seul signal.

## Les niveaux

| Niveau | Quand | Flutter (`HapticFeedback`) | iOS (`UIFeedbackGenerator`) | Android (`HapticFeedbackConstants`) |
|---|---|---|---|---|
| 1. Sélection | choisir une option, une puce, un onglet, faire défiler un sélecteur | `selectionClick` | `UISelectionFeedbackGenerator` | `CLOCK_TICK` / `SEGMENT_TICK` |
| 2. Geste confirmé | cocher, valider une petite action, poser un élément | `lightImpact` | `UIImpactFeedbackGenerator(.light)` | `CONFIRM` / `VIRTUAL_KEY` |
| 3. Étape enregistrée | enregistrer, envoyer, terminer une étape d'un parcours | `mediumImpact` | `UIImpactFeedbackGenerator(.medium)` | `CONFIRM` |
| 4. Célébration | fin d'un parcours, réussite, récompense | motif de succès (deux impacts courts) | `UINotificationFeedbackGenerator(.success)` | `CONFIRM` répété |
| 5. Alerte | erreur, valeur refusée, action urgente, suppression | motif d'avertissement (`heavyImpact` ou deux impacts) | `UINotificationFeedbackGenerator(.warning / .error)` | `REJECT` |

- Pas de vibration pour un simple défilement, une navigation ordinaire, ni une animation qui se déclenche toute seule.
- Une vibration par geste : jamais plusieurs à la suite pour la même action, jamais en boucle.

## Mise en œuvre

- **Un service unique** appelé par intention (`haptics.selection()`, `confirm()`, `saved()`, `celebrate()`, `alert()`) : jamais d'appel direct à l'API de la plateforme dans les écrans. Le service applique le niveau et respecte les réglages.
- Le service est **injecté**, ce qui permet de le remplacer par un double dans les tests. Chaque geste branché a un test qui vérifie le niveau émis.
- Appel au moment où l'action est **effectivement prise en compte** (après validation, pas au simple appui si l'action peut être refusée).
- Échec ou absence de moteur de vibration : silencieux, sans erreur ni journal d'erreur.

## Réglages et accessibilité

- Un interrupteur **« Vibrations »** dans les réglages de l'app, activé par défaut, qui coupe tous les retours haptiques.
- Respecter les réglages du système (vibrations du téléphone coupées, mode silencieux selon la plateforme).
- Le retour haptique n'est pas une animation : « Réduire les animations » ne le coupe pas. Seul l'interrupteur « Vibrations » le fait.

## Web et bureau

- Web : `navigator.vibrate` seulement pour les niveaux 4 et 5, et seulement sur mobile, derrière la même préférence. Bureau : aucun retour haptique.
