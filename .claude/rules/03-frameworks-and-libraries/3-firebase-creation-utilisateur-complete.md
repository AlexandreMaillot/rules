# Firebase — création d'utilisateur = Firestore + Auth + mail reset (par défaut)

## Principe (obligation par défaut)

Dès qu'une fonctionnalité **crée un utilisateur** et que le projet s'appuie sur
**Firebase**, si l'intention n'est **pas explicitement précisée autrement**, la
création DOIT faire les **trois** choses ensemble, dans le même flux :

1. **Firestore** — écrire le(s) document(s) métier de l'utilisateur (profil, doc
   applicatif, rattachement, rôle…).
2. **Firebase Auth** — créer (ou rattacher) le **compte Auth** via l'**Admin SDK**
   côté serveur (`createUser`, ou résolution « ou rattacher » si l'email existe
   déjà). Jamais côté client pour une création pilotée par un back-office.
3. **Email de réinitialisation de mot de passe** — générer le lien via l'Admin
   SDK (`generatePasswordResetLink`) **puis l'ENVOYER**. Générer le lien ne
   suffit pas : sans envoi, l'utilisateur ne reçoit rien.

Ne jamais livrer une création d'utilisateur qui s'arrête à Firestore, ou qui
crée le compte Auth sans jamais délivrer le moyen de définir le mot de passe.

## Envoi du mail

Réutiliser le transport transactionnel existant du projet — ne pas introduire un
nouveau fournisseur. Pattern courant Firebase : écrire un document dans une
collection `mail` consommée par l'extension **« Trigger Email »
(`firestore-send-email`)**. Template HTML brandé, échappement anti-injection du
nom et du lien.

## Best-effort sur l'email (ne pas casser la création)

L'envoi du mail est **best-effort** : si le transport échoue, la création
(Firestore + Auth) **reste valide** — remonter un flag (ex. `lienResetEchoue`)
pour permettre une relance, sans faire échouer l'opération. Catch non
silencieux + log.

## Cas « compte déjà existant »

Si l'email correspond à un compte Auth déjà existant (rattachement), **ne pas**
renvoyer de mail de réinitialisation : l'utilisateur a déjà un mot de passe. Le
mail reset ne concerne que les **comptes réellement créés**.

## Bypass acceptables (précisés explicitement)

- Onboarding différent demandé (mot de passe pré-défini, lien magique, pas
  d'email).
- Import/seed technique de comptes (script admin).
- Auto-inscription publique (l'utilisateur choisit lui-même son mot de passe).

Dans ces cas, **documenter** le choix dans le code / la PR.

## Checklist à l'ajout d'un flux de création d'utilisateur

- [ ] Doc(s) Firestore écrit(s).
- [ ] Compte Firebase Auth créé/rattaché via Admin SDK (serveur).
- [ ] Lien reset généré ET envoyé (collection `mail` / Trigger Email).
- [ ] Best-effort mail (flag d'échec, pas de throw bloquant).
- [ ] Compte existant → pas de mail reset.
