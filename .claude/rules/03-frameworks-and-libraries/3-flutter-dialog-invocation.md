---
description: TOUJOURS utiliser `await` avec `showDialog`, `showModalBottomSheet` et autres méthodes de navigation asynchrones.
globs: apps/*/lib/**/*.dart
alwaysApply: true
---

Principe:

- Les méthodes de navigation asynchrones Flutter retournent des Futures
- Utiliser `await` garantit l'ordre d'exécution et permet de récupérer le résultat (si présent)
- Sans `await`, le code continue immédiatement sans attendre la fermeture du dialogue
- Même sans valeur de retour, `await` est obligatoire pour garantir l'ordre d'exécution

Règle absolue:

- OBLIGATOIRE : Toujours utiliser `await` devant `showDialog`, `showModalBottomSheet`, `showGeneralDialog`, etc.
- OBLIGATOIRE : Même si on n'utilise pas la valeur de retour, `await` est obligatoire pour l'ordre d'exécution
- OBLIGATOIRE : La méthode appelante doit être `async`
- OBLIGATOIRE : Vérifier `context.mounted` après un `await` si on utilise le contexte

Méthodes concernées:

- `showDialog<T>()` - Dialogues modaux
- `showModalBottomSheet<T>()` - Bottom sheets modaux
- `showGeneralDialog<T>()` - Dialogues personnalisés
- `showDatePicker()` - Sélecteur de date
- `showTimePicker()` - Sélecteur d'heure
- Toutes les méthodes de navigation qui retournent un `Future<T>`

Exemples interdits:

```dart
// ❌ showDialog sans await (même sans valeur de retour)
void _onSupprimer() {
  showDialog<void>(
    context: context,
    builder: (_) => const ConfirmationDialog(),
  );
  // Le code continue immédiatement, le dialogue peut ne pas être encore affiché
  _supprimer(); // Exécuté avant la fermeture du dialogue
}
```

Exemples corrects:

```dart
// ✅ showDialog avec await et valeur de retour
Future<void> _onSupprimer() async {
  final confirme = await showDialog<bool>(
    context: context,
    builder: (_) => const ConfirmationDialog(),
  );

  if (!context.mounted) return;

  if (confirme == true) {
    _supprimer();
  }
}

// ✅ showDialog avec await SANS valeur de retour (await obligatoire quand même)
Future<void> _onAfficherInfo() async {
  await showDialog<void>(
    context: context,
    builder: (_) => const InfoDialog(),
  );

  if (!context.mounted) return;

  _apresFermetureDialogue();
}
```

Cas spéciaux:

- Même sans valeur de retour, `await` est OBLIGATOIRE : `await showDialog<void>(...)`
- L'`await` garantit que le code suivant s'exécute APRÈS la fermeture du dialogue
- Toujours vérifier `context.mounted` après un `await` si on utilise le contexte après
- Pour les dialogues de confirmation, utiliser `Future<bool?>` pour récupérer le choix de l'utilisateur

Bonnes pratiques:

- Toujours typer le Future retourné : `Future<T> showDialog<T>(...)`
- Utiliser des valeurs de retour explicites (bool, String, etc.) plutôt que void quand c'est pertinent
- Vérifier la nullité du résultat avant de l'utiliser
- Gérer les cas où l'utilisateur ferme le dialogue sans action (retourne null)

Conséquences du non-respect:

- Bugs de timing : code exécuté avant la fermeture du dialogue
- Impossible de récupérer les valeurs de retour
- Comportements imprévisibles et difficiles à déboguer
- Risque d'utiliser un contexte invalide après navigation

## Cas spécial — Dialog piloté par un state bloc

Si l'ouverture / fermeture du dialog est **pilotée par un state bloc** (pattern `dialogXxx: Entite?` dans l'état, ouverture par `BlocListener.listenWhen`), la fermeture **barrier-dismiss** ou **ESC** ne passe pas par les callbacks utilisateur → le state reste "dialog ouvert" et le prochain clic sur le déclencheur **ne rebuild pas** le listener (même valeur émise).

**Obligatoire** : après `await showDialog(...)`, émettre un event de fermeture idempotent pour forcer le reset du state :

```dart
// ✅ Reset défensif — couvre barrier-dismiss + ESC + back button
Future<void> _afficherDialogueSuppression(BuildContext context, Entite e) async {
  await DialogueSuppression.afficher(
    context: context,
    libelle: e.libelle,
    onConfirmer: () => context.read<MonBloc>().add(SuppressionConfirmee(e.id)),
    onAnnuler: () => context.read<MonBloc>().add(const SuppressionAnnulee()),
  );
  if (context.mounted) {
    context.read<MonBloc>().add(const SuppressionAnnulee());
  }
}
```

**Alternative non recommandée** : écouter `Navigator.didPop` pour router tous les cas — trop couplé, fragile en cas de multi-dialogs imbriqués.
