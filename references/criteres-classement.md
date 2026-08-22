# Critères de classement

Chargé en phase 3, quand un bloc résiste au tableau du `SKILL.md`. Les quatre étiquettes sont
exclusives : un bloc en reçoit une, jamais deux.

## Le test, dans l'ordre

Pose les questions dans cet ordre et arrête-toi à la première qui tranche.

1. **Est-ce vrai ?** Le verdict de la phase 2. Si c'est `NON VÉRIFIABLE`, le bloc reste où il est,
   tu ne le classes pas. Tu le signales dans le rapport, sous une rubrique à part.
2. **Est-ce faux ?** Regarde s'il reste quelque chose à garder. Une règle fausse dont la valeur se
   corrige garde son étiquette d'avant, tu changes juste la valeur. Une règle fausse qui trace une
   décision passée va en `ARCHIVE`. Une règle fausse qui ne laisse rien va en `SUPPRIMER`.
3. **Est-ce que ça vaut partout, tout le temps ?** Oui, c'est `RACINE`. Non, c'est `DOC`.

## RACINE

Vrai, et vrai quel que soit le fichier sur lequel on travaille. C'est la seule catégorie qui coûte
des tokens à chaque session, donc elle se mérite.

Y vont : ce qu'est le projet en deux phrases, les commandes de build, de test et de lint, les
interdits sans exception, les liens vers les fiches.

```markdown
Ne jamais commiter sur `main`. Le travail passe par une branche et une PR.
```

Vrai partout, aucune exception, une ligne. C'est le format cible.

Un doute utile : si tu peux nommer un dossier ou une tâche où la règle ne s'applique pas, ce n'est
pas `RACINE`.

## DOC

Vrai, mais qui ne sert qu'à celui qui travaille sur un sujet précis. Un module, un dossier, une
procédure, une intégration.

```markdown
Les migrations Alembic se génèrent avec `alembic revision --autogenerate`, puis se relisent
à la main : l'autogénération rate les renommages de colonne et les transforme en drop plus add.
```

Utile, précis, vrai. Et inutile pour les neuf dixièmes des sessions. Ça part dans
`docs/migrations.md`, avec un lien depuis la racine.

Avant de créer la fiche, vérifie qu'elle n'existe pas déjà. Un contenu qui redit un `README.md`
ou une fiche existante devient un lien vers elle, pas un fichier de plus.

Deux blocs `DOC` sur le même sujet vont dans la même fiche. Une fiche par bloc, c'est un
éparpillement, et personne ne retrouve rien.

## ARCHIVE

Était vrai. Ne l'est plus. Et garde une valeur de trace, parce que quelqu'un se posera la question
« pourquoi c'était comme ça ».

```markdown
Le cache tourne sur Redis. Ne pas mettre de clé sans TTL, l'instance est petite.
```

Si le projet est passé sur autre chose, la règle est fausse, mais elle explique pourquoi il reste
du code de sérialisation bizarre. Elle part dans `archives/`, avec un en-tête qui donne la date
d'archivage et la raison en une ligne.

Le critère qui sépare `ARCHIVE` de `SUPPRIMER` : est-ce que ça répond à une question que
quelqu'un se posera en lisant le code ? Oui, on archive. Non, on supprime.

Une règle fausse depuis longtemps que personne n'a jamais appliquée n'est pas une trace, c'est du
bruit. Elle va en `SUPPRIMER`.

## SUPPRIMER

Faux sans rien à garder, ou vrai mais vide de sens. Le détail des formes vides est dans
`anti-patterns.md`.

La suppression est la seule étiquette qui exige une ligne dans le tableau final du rapport, avec
sa preuve à côté. C'est ce qui permet à l'opérateur de refuser au cas par cas.

## Les cas limites

| Cas | Ce que tu fais |
|---|---|
| Une règle vraie à 60 % | tu ne classes pas, tu poses la question en phase 4 avec le comptage |
| Une règle qui répète une autre règle du même fichier | tu gardes la formulation la plus précise, l'autre va en `SUPPRIMER` |
| Une règle vraie mais qu'un fichier imbriqué contredit | contradiction, phase 4 |
| Une règle vraie qui ne concerne qu'un seul fichier | `DOC`, et le lien va dans un commentaire de ce fichier plutôt qu'à la racine |
| Un bloc qui mélange une règle `RACINE` et un détail `DOC` | tu le coupes en deux, chaque morceau prend son étiquette |
| Une règle de sécurité, même invérifiable | elle reste. On ne déplace pas un garde-fou sur une impression |
| Une règle qui vient de la portée utilisateur | tu ne la touches pas, tu la signales |
