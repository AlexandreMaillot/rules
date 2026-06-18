---
description: Quand plusieurs agents de code (ou tout agent qui modifie le working tree) tournent en parallèle sur le même repo, chacun DOIT utiliser un `git worktree add` dédié dès le 1er fichier édité. Un `git checkout` dans le main repo efface silencieusement les working-tree edits des autres agents.
globs:
  - "*"
alwaysApply: false
---

# Multi-agent — worktree dédié obligatoire pour tout agent qui édite

## Pourquoi

Lorsqu'un agent (ou tout sous-agent qui modifie des fichiers) bascule de branche dans le main repo, Git :

1. Vide le working tree de tous les fichiers modifiés (non commités)
2. Repose les fichiers de la branche cible
3. Les modifications working-tree de l'autre agent parallèle sont **silencieusement perdues** (pas d'erreur, pas de warning)

Détecté via `git reflog` après coup — mais entre-temps des heures de travail peuvent être effacées.

## Règle absolue

**Tout agent qui modifie le working tree** (édition de fichier, commit, rebase, cherry-pick, stash, checkout) en parallèle d'un autre agent **DOIT** :

1. **Première action** : `git worktree add /tmp/<agent-name>-<sujet> <branche-ou-master>` (puis `cd /tmp/<agent-name>-<sujet>`)
2. **INTERDIT** : tout `git checkout`, `git stash`, `git rebase`, `git reset`, `git cherry-pick` dans le main repo tant qu'un autre agent y travaille
3. **Sous-worktrees parallélisables** : si l'agent lance lui-même 2-3 PRs en parallèle, créer 2-3 sous-worktrees (`/tmp/<agent>-pr1`, `/tmp/<agent>-pr2`, ...)
4. **Nettoyage** : `git worktree remove --force /tmp/<agent>-<sujet>` après merge complet (sinon les branches restent verrouillées par le worktree)

## Comment briefer un agent en parallèle

Inclure systématiquement dans le brief :

> ⚠️ CONTRAINTE worktree dédié obligatoire
>
> Un autre agent tourne en parallèle sur le main repo. INTERDICTION absolue d'y toucher.
>
> 1. Première action : `git worktree add /tmp/<sujet>-<phase> master` puis `cd /tmp/<sujet>-<phase>`
> 2. Pour stack interne : sous-worktrees `/tmp/<sujet>-pr1` + `/tmp/<sujet>-pr2`
> 3. Nettoie après merge : `git worktree remove --force`

## Anti-patterns

- ❌ « Je fais juste un `git status` rapide dans le main repo » — on prend l'habitude. Toujours via worktree.
- ❌ Lancer un agent en background, puis faire des `git checkout` manuels dans le main repo pour vérifier l'état d'une PR → l'agent perd ses edits.
- ❌ Brief sans contrainte worktree quand ≥ 2 agents en parallèle.

## Workflow recommandé (template brief)

```
1. git worktree add /tmp/<sujet>-<phase> master
2. cd /tmp/<sujet>-<phase>
3. git checkout -b feat/<branche>
4. <code + tests + commits + push>
5. gh pr create
6. <attendre merge ou retry>
7. cd <main-repo>  (revenir au main repo HORS edits)
8. git worktree remove --force /tmp/<sujet>-<phase>
```
