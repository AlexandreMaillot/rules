---
description: PRs stackées et agents — éviter de casser une PR dépendante au squash-merge, et ne pas perdre les modifs non-commitées du working tree quand un agent change de branche.
globs:
alwaysApply: true
---
## PRs stackées (PR B basée sur PR A)

- **Ne pas squash-merger** une PR (A) qui sert de base à une autre (B) sans préparer B : le squash crée un nouveau commit sur `master`, la branche de B reste sur l'ancien tip de A → diff bruité / conflits, et GitHub **ferme automatiquement** B quand la branche de base est supprimée (`--delete-branch`).
- Workflow correct quand on merge A avec squash :
  1. merger A (`gh pr merge <A> --squash --delete-branch`),
  2. rebaser B sur le nouveau `master` en droppant les commits de A : `git fetch && git rebase --onto origin/master <ancien-tip-de-A> <branche-B>` (ou `git rebase origin/master <branche-B>` si A n'avait qu'un commit content-identique), `git push --force-with-lease`,
  3. si B a été fermée → en **recréer une** (`gh pr create --base master --head <branche-B>`), `gh pr reopen`/`gh pr edit --base` ne marchent pas sur une PR fermée dont la branche de base a disparu.
- Alternative : merger A avec `--merge` (pas `--squash`) — les stacks restent intactes.

## Modifs non-commitées + agents qui changent de branche

- **Toujours committer (ou `git stash`) les modifications du working tree AVANT de lancer un agent qui fait des `git checkout` / `git rebase`** — sinon elles peuvent être perdues (vécu : des fixes UI ont disparu pendant des changements de branche d'agents, à reconstruire).
- Après un agent qui a touché git : `git status` pour vérifier l'état du working tree avant tout commit.
- Quand un agent committe : `git add <fichiers précis>`, **jamais `git add -A`** (sinon des fichiers non liés — `.specstory/`, `.idea/runConfigurations/*`, fichiers générés d'autres apps — partent dans le commit).
- Le `cd` du shell persiste entre les commandes : après un `cd /tmp/<autre-repo>` (ex. clone d'un repo de référence), **revenir explicitement au repo principal** avant les commandes `git`/`gh`, sinon elles ciblent le mauvais dépôt.

## Déploiement après merge

- PR qui modifie `firestore.rules` → `firebase deploy --only firestore:rules` après le merge (cf. `3-firestore-deploy-rules-apres-merge.mdc`).
- PR qui modifie l'API → redéployer le runtime serveur (Cloud Run) pour que le changement soit servi.
