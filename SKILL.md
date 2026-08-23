---
name: clawd-audit-md-agent
description: "Raccourcit le CLAUDE.md ou l'AGENTS.md d'un projet en sortant le détail vers des fiches liées ou des skills, pour que la racine ne porte plus que ce qui doit être vrai à chaque session. Confronte chaque ligne au code réel avant de la déplacer, avec une preuve à côté de chaque coupe, et ne touche pas à ce qu'il ne peut pas vérifier. Traite aussi les fichiers de sous-agents et le catalogue de skills. Lit la mémoire de l'agent sans jamais y écrire. N'écrit rien tant que le rapport n'a pas été validé. Se déclenche sur : mes instructions sont trop longues, optimise mon CLAUDE.md, allège mon AGENTS.md, audit des instructions, sortir le contenu vers docs, est-ce que les règles sont encore vraies, instructions obsolètes, ranger mes skills, réduire le contexte chargé à chaque session."
version: 1.1.1
author: fullya99
license: MIT
platforms: [linux, macos, windows]
compatibility: "Aucune dépendance externe. Les deux scripts sont du bash POSIX et n'utilisent que find, grep, sed et awk. Aucun accès réseau."
metadata:
  hermes:
    tags: [audit, instructions, documentation, context, cleanup, skills]
---

# Clawd audit md, pour un agent permanent

> **Le contrat** : un fichier racine court, où chaque ligne restante est vraie et prouvée, et tout
> le reste rangé là où ça ne coûte rien tant que ça ne sert pas. Rien n'est écrit avant validation,
> et jamais une ligne dans ta propre mémoire.

**Le but est de raccourcir.** Un fichier d'instructions grossit tout seul, personne ne relit les
lignes du haut, et il finit par décrire un projet qui n'existe plus. Il est chargé en entier au
début de chaque session, donc chaque ligne est payée pour toujours, qu'elle serve ou non.

La bonne image : ce fichier est de la mémoire vive, pas du disque. Presque tout ce qu'il contient
peut vivre ailleurs et ne charger que quand ça sert, dans un skill ou dans une fiche liée. Il y a
quatre étages, ils ne se valent pas, et choisir le bon est le vrai travail. C'est dans
`references/les-quatre-etages.md`.

**La vérification sert à ça, elle n'est pas le but.** On ne déplace pas une ligne qu'on n'a pas
comprise, et on ne supprime pas une ligne fausse sans montrer en quoi elle l'est.

**Avant d'écrire quoi que ce soit**, charge le skill de style s'il est installé. Sinon, le minimum
tient en une ligne : français, ton direct, aucun tiret cadratin, aucun point-virgule, aucune
tournure qui sent le texte généré.

---

## Quand ce skill s'applique

| Situation | Ce que tu fais |
|---|---|
| « c'est trop long », « allège », « optimise mes instructions » | les six phases, en entier. C'est le cas courant |
| « audite mes instructions », « c'est faux » | pareil, les six phases. Le tri produit le raccourcissement |
| « vérifie juste si c'est encore vrai » | phases 1 et 2, tu rends le tableau des preuves et tu t'arrêtes |
| « sors ça dans docs/ » sur un bloc précis | phases 3 et 5 sur ce bloc, la preuve reste obligatoire |
| « range mes skills » | l'annexe catalogue en fin de fiche, pas les six phases |
| Le projet n'a aucun fichier d'instructions | tu ne fais rien, tu le dis. En écrire un de zéro est un autre travail |
| On te demande d'ajouter une règle | hors périmètre. Ce skill déplace et supprime, il n'invente pas |

---

## Deux choses à savoir avant tout le reste

**Tu tournes en permanence, mais ta session repart à zéro.** Un `/new` chez Hermes, une session
neuve chez OpenClaw, et ce que tu avais compris a disparu. Un audit qui s'étale sur deux sessions
perd son tableau de preuves entre les deux. Fais-le d'un bloc, ou écris le rapport dans un fichier
du projet avant de t'arrêter.

**Tu n'as pas de répertoire courant fiable.** Ton cwd persiste entre les appels et ne désigne rien
en particulier. Un `find .` lancé depuis ton dossier personnel balaierait toute la machine. Alors
la première chose que tu fais, c'est établir sur quel projet tu travailles, et le dire.

```bash
pwd && git rev-parse --show-toplevel 2>/dev/null || echo "pas un dépôt git"
```

Si l'opérateur n'a pas nommé le projet et que le cwd n'est pas un dépôt, tu demandes. Tu ne
devines pas.

---

## Sur quel lecteur tourne ce projet

Les fichiers d'instructions ne portent pas le même nom selon le lecteur, et un même dépôt en a
souvent plusieurs.

| Lecteur | Dans le dépôt | Hors dépôt, à signaler sans y toucher |
|---|---|---|
| Hermes Agent | `AGENTS.md`, `.hermes.md`, `HERMES.md`, et il lit aussi `CLAUDE.md` | `~/.hermes/SOUL.md`, `~/.hermes/memories/` |
| OpenClaw | `AGENTS.md` du workspace, `skills/*/SKILL.md` | `SOUL.md`, `USER.md`, `MEMORY.md`, `memory/AAAA-MM-JJ.md` |
| Claude Code | `CLAUDE.md`, `.claude/CLAUDE.md`, `CLAUDE.local.md`, les imbriqués, `.claude/rules/*.md`, `.claude/agents/*.md` | `~/.claude/` |
| Codex, standard ouvert | `AGENTS.md` et ses imbriqués, `.agents/skills/` | `~/.agents/`, `~/.codex/` |
| Cursor, Windsurf, Cline | `.cursor/rules/*.mdc`, `.windsurfrules`, `.clinerules` | hors dépôt |

**Claude Code ne lit pas `AGENTS.md`.** C'est le piège le plus courant. Un dépôt qui porte les deux
fichiers fait tourner deux sources qui divergent, sans que rien ne le dise. Le raccord se fait par
une ligne `@AGENTS.md` en tête du `CLAUDE.md`, ou par un lien symbolique. Un dépôt qui a les deux
sans raccord est une constatation de phase 2, pas une opinion.

**Tu ne traites aucune instruction comme universelle.** `/clear` et `/compact` sont des cas Claude
Code, `/new` est le tien. Une règle qui en parle reste vraie, mais elle se range en disant pour
quel lecteur elle vaut.

### Les imports

Chez Claude Code, une ligne `@chemin/vers/fichier` en dehors d'un bloc de code charge ce fichier au
démarrage, récursivement, jusqu'à quatre sauts. Un import qui ne résout pas est une ligne morte, et
rien ne le signale. Un fichier importé compte dans le budget de contexte comme s'il était collé
dans le fichier racine, donc découper en imports n'allège rien. Un chemin entre accents graves
n'est pas un import, c'est du texte.

---

## Ta mémoire à toi, tu la lis, tu n'y touches pas

C'est la règle la plus importante de ce skill, et elle n'a pas d'exception.

| Fichier | Ce que tu fais |
|---|---|
| `SOUL.md` | tu lis, tu signales une contradiction avec le projet, tu n'écris jamais |
| `USER.md` | pareil |
| `MEMORY.md` | pareil |
| `memory/AAAA-MM-JJ.md` | pareil |
| la config de la plateforme | tu ne la lis même pas |

Deux raisons, et la seconde est la vraie.

Un audit qui corrige ta propre mémoire n'a plus de juge extérieur. Tu deviendrais à la fois
l'auteur de la règle, celui qui la vérifie et celui qui la réécrit. Le rapport ne veut plus rien
dire.

Et l'écriture en mémoire d'agent est le premier vecteur d'attaque documenté de cet écosystème.
C'est pour ça que les scanners d'installation la classent critique. Ce skill n'y écrit pas, et
n'apprend à personne comment le faire.

Ce que tu peux faire, et qui est utile : dire à l'opérateur qu'une ligne de sa mémoire contredit
une ligne du dépôt, montrer les deux, et le laisser trancher. Le détail est dans
`references/memoire-de-l-agent.md`.

---

## Avant de commencer, deux barrières

**L'arbre git doit être propre.** C'est ce qui permet d'annuler tout d'un `git checkout`.

```bash
git status --porcelain
```

Si ça sort quelque chose, tu t'arrêtes et tu demandes de commiter d'abord. Pas de sauvegarde
maison, pas de copie `.bak`, pas de stash décidé tout seul.

Si le projet n'est pas un dépôt git, tu le dis, et tu demandes une autorisation explicite avant
d'écrire quoi que ce soit. Sans historique, une erreur de ta part n'est pas réparable.

**La convention de rangement du projet gagne.**

```bash
ls -1 docs/ archives/ CODEMAP.md TODOS.md CHANGELOG.md README.md 2>/dev/null
```

Si `docs/` et `archives/` sont là, tu ranges dedans. Si le projet a son propre système, tu suis le
sien. Si rien n'existe, tu proposes `docs/` et `archives/` et **tu attends la confirmation**.

---

## Les six phases

### Phase 1. Inventaire

```bash
find . -path ./node_modules -prune -o \( -name 'CLAUDE.md' -o -name 'CLAUDE.local.md' \
  -o -name 'AGENTS.md' -o -name '.hermes.md' -o -name 'HERMES.md' \
  -o -name '.clinerules' -o -name '.windsurfrules' \
  -o -path '*/.claude/rules/*.md' -o -path '*/.claude/agents/*.md' \
  -o -path '*/.cursor/rules/*' \) -print 2>/dev/null

for f in $(find . -name 'CLAUDE.md' -o -name 'AGENTS.md' 2>/dev/null); do
  printf '%s\t%s lignes\t%s\n' "$f" "$(wc -l < "$f")" "$(git log -1 --format=%ad --date=short -- "$f")"
done

grep -nE '^[[:space:]]*@[^[:space:]`]+' CLAUDE.md 2>/dev/null   # les imports
```

Pour chacun : taille en lignes, date du dernier commit, portée de chargement. Un fichier imbriqué
ne se charge que dans son dossier. Un fichier importé coûte comme s'il était collé dans la racine,
donc **sa taille compte dans le total**.

Relève aussi ce qui existe déjà comme documentation dédiée, `docs/`, `README.md`, les décisions.
Une ligne dont le contenu est déjà écrit ailleurs ne part pas en fiche, elle devient un lien.

### Phase 2. Vérification contre le code

C'est ce qui rend les coupes défendables. Lance d'abord le contrôle mécanique.

```bash
bash "$SKILL/scripts/verif-refs.sh" CLAUDE.md
bash "$SKILL/scripts/verif-refs.sh" --format=md AGENTS.md   # tableau markdown, pour le rapport
```

`$SKILL` est le répertoire de ce skill. Tu travailles depuis le projet, pas depuis le skill, donc
`scripts/verif-refs.sh` tout court ne résout pas. Résous-le une fois, avant le premier appel :

```bash
ANCETRES="$(d="$PWD"; while [ "$d" != "/" ]; do echo "$d/.agents/skills/clawd-audit-md-agent"; d="$(dirname "$d")"; done)"

SKILL="$(for d in \
  $(find "$HOME/.hermes/skills" -maxdepth 3 -type d -name 'clawd-audit-md-agent' 2>/dev/null) \
  $(find "$HOME/.openclaw" -maxdepth 5 -type d -name 'clawd-audit-md-agent' 2>/dev/null) \
  "skills/clawd-audit-md-agent" ".agents/skills/clawd-audit-md-agent" $ANCETRES \
  "$HOME/.agents/skills/clawd-audit-md-agent" \
  "$HOME/.codex/skills/clawd-audit-md-agent" \
  "$HOME/.claude/skills/clawd-audit-md-agent"; do
  [ -f "$d/scripts/verif-refs.sh" ] && echo "$d" && break
done)"
```

L'ordre suit les portées de chaque plateforme. Chez Hermes, `~/.hermes/skills/`, avec ou sans
dossier de catégorie, le scan est récursif. Chez OpenClaw, le workspace d'abord, puis
`.agents/skills/` du projet, puis `~/.agents/skills/`. L'entrée relative `skills/<nom>` sert au
workspace OpenClaw déplacé, qui est le seul répertoire de travail de l'agent. Le test porte sur un
fichier que seul ce skill possède, ce qui écarte les homonymes. Si la résolution échoue, tu fais
les vérifications à la main, le script fait gagner du temps mais il n'est pas la règle.

Le script ne couvre pas tout. Ce qui reste à ta charge :

| Affirmation | Comment tu la vérifies |
|---|---|
| Une dépendance, un framework, une version | tu lis le manifeste réel et tu compares la valeur, pas juste le nom |
| Un `paths:` de règle, un glob | il doit matcher au moins un fichier, sinon la règle est morte |
| Une convention de code | tu échantillonnes et tu comptes. « 3 fichiers sur 40 » est la preuve |
| Une règle d'organisation | un grep ciblé qui la contredirait. S'il ne sort rien, c'est confirmé |

Trois verdicts, et un seul par affirmation.

- **`VRAI`** : le dépôt confirme.
- **`FAUX`** : le dépôt contredit, **et tu tiens la preuve**. Une preuve, c'est quelque chose que
  l'opérateur peut relancer.
- **`NON VÉRIFIABLE`** : tu n'as pas de quoi trancher. C'est un verdict honnête, pas un échec. Une
  affirmation invérifiable **n'est jamais marquée fausse par défaut** et **reste où elle est**.

Le taux de respect compte. Une convention suivie à 90 % est vraie. À 8 %, elle est fausse, et la
preuve est le comptage. Entre les deux, tu poses la question en phase 4 au lieu de trancher.

### Les fichiers d'agents

Un dépôt qui déclare des sous-agents porte un second jeu de fichiers d'instructions, et celui-là
casse en silence. Un fichier mal formé n'est pas chargé, sans erreur et sans avertissement.

```bash
bash "$SKILL/scripts/verif-agents.sh" .claude/agents/
```

Le détail est dans `references/fichiers-agents.md`. **Tu ne réécris jamais le corps d'un fichier
d'agent.** Le frontmatter se corrige, le prompt est un choix de son auteur.

### Phase 3. Classement, bloc par bloc

Un bloc, c'est une règle, pas une ligne. Chaque bloc reçoit exactement une étiquette.

| Étiquette | Critère | Destination |
|---|---|---|
| `RACINE` | vrai, et vrai partout dans le dépôt, tout le temps | reste dans le fichier racine |
| `SORTIR` | vrai, mais ne sert que pour une tâche, un dossier ou un module | un skill, ou une fiche liée |
| `ARCHIVE` | était vrai, et garde une valeur de trace | `archives/`, avec sa date et la raison |
| `SUPPRIMER` | faux sans rien à garder, ou vide de sens | supprimé, listé dans le rapport avec la preuve |

**`SORTIR` est l'étiquette qui fait le travail**, et elle demande un second choix. Un skill charge
son corps seulement quand il sert, et sa description reste visible, donc il peut se déclencher
tout seul. Une fiche liée ne charge que si tu décides d'ouvrir le lien, donc elle ne garantit rien.

La règle qui tranche : **une contrainte qui doit être appliquée ne va jamais dans une fiche.** Elle
va en skill. La fiche porte ce qu'on consulte, du contexte, une décision passée, un piège
documenté. Et si aucune sortie ne tient la contrainte, elle reste à la racine, même si ça allonge.

Chez Claude Code il existe un étage de plus, la règle liée à des chemins, qui se charge toute seule
quand l'agent touche un fichier qui matche. Tu ne l'as pas, mais si tu audites un dépôt lu par
Claude Code, c'est souvent la meilleure sortie et tu la proposes. Tout est dans
`references/les-quatre-etages.md`. Les cas limites du classement sont dans
`references/criteres-classement.md`, les formes de règles inutiles dans
`references/anti-patterns.md`.

Une règle fausse qui redevient vraie en corrigeant une valeur n'est pas à supprimer. Tu corriges la
valeur, tu gardes la règle, et tu montres l'ancienne et la nouvelle dans le rapport.

### Phase 4. Contradictions

Dresse la liste des règles qui s'opposent. Deux blocs qui disent l'inverse, un fichier imbriqué qui
contredit la racine, une règle du dépôt que ta mémoire contredit.

Pour chacune, **tu poses la question**. Tu montres les deux versions, leur emplacement et leur
date, et tu t'arrêtes là. **Tu ne tranches jamais tout seul**, même quand une version semble
évidente. Celle qui a l'air périmée est parfois la seule que l'équipe applique.

### Phase 5. Restructuration

Le fichier racine vise **50 lignes**, plafond dur **100**. Il porte ce qu'est le projet en deux
phrases, les commandes réelles, les contraintes sans exception, et la table de renvoi.

**La table de renvoi est ce qui fait tenir les 50 lignes.** Chaque ligne dit la situation avant de
donner le chemin, sinon elle ne se fait pas ouvrir.

```markdown
| Quand | Où |
|---|---|
| Tu ajoutes ou modifies une migration | `docs/migrations.md` |
```

Chaque fiche commence par dire quand elle s'applique, puis donne les règles. Une fiche traite un
sujet. Au-delà de 150 lignes, coupe en deux. En dessous de 15, ne la crée pas, ces lignes rejoignent
une fiche voisine ou restent à la racine.

**Un `@import` n'allège rien**, il colle le contenu dans le fichier racine au démarrage. Si le
fichier audité en contient, dis-le. Les gabarits sont dans `references/modeles.md`.

**Tu ne casses pas les liens.** Quand un contenu se déplace, tu repasses sur tout le dépôt et tu
corriges les références vers son ancien emplacement, puis tu relances le script sur le résultat.

```bash
grep -rn "CLAUDE.md\|AGENTS.md" --include='*.md' --include='*.yml' . | grep -v node_modules
bash "$SKILL/scripts/verif-refs.sh" CLAUDE.md
```

### Phase 6. Rapport, puis validation

Tu présentes le plan complet **avant d'écrire quoi que ce soit**. Quatre morceaux, dans cet ordre.

1. Le tableau des vérifications, une ligne par affirmation, avec le verdict et la preuve.
2. Le diff du fichier racine, avec son compte de lignes avant et après, et le gain en pourcentage.
3. La liste de ce qui sort, avec pour chaque morceau son étage et son nombre de lignes.
4. Le tableau des suppressions, une ligne par bloc, avec la preuve.

Puis tu attends. L'opérateur approuve, corrige, ou refuse bloc par bloc. **Tu n'écris qu'après.**

Une fois l'écriture faite, relance l'audit. Un deuxième passage sur un projet déjà traité ne doit
rien changer.

---

## Annexe. Ranger le catalogue de skills

Demande à part, pas une des six phases. Elle vaut chez Hermes, qui écrit ses propres skills et les
retouche à l'usage, et chez OpenClaw, dont l'atelier accumule des propositions.

Le problème est le même qu'un fichier d'instructions qui enfle : personne ne relit, et le catalogue
finit avec des doublons et des skills qui décrivent un outil disparu.

```bash
for f in $(find "$HOME/.hermes/skills" "$HOME/.openclaw" -name SKILL.md 2>/dev/null); do
  printf '%s\t%s lignes\n' "$f" "$(wc -l < "$f")"
done
```

Quatre constats, et tu les rends sans rien supprimer.

- **Deux descriptions qui se recouvrent.** Le mauvais skill gagne une fois sur deux. Tu montres la
  paire, l'opérateur tranche.
- **Un skill dont le binaire ou le chemin n'existe plus.** Passe le script dessus, la preuve sort
  toute seule.
- **Un skill jamais déclenché.** Sa description ne dit pas quand l'appeler. Tu proposes une
  reformulation, tu ne l'appliques pas.
- **Un `SKILL.md` de plus de 500 lignes.** Il est payé à chaque chargement. Le détail sort en
  `references/`.

**Tu ne supprimes aucun skill toi-même**, y compris ceux que tu as écrits. Tu listes, tu prouves,
l'opérateur décide.

---

## Invariants, jamais négociables

- ❌ **Aucune écriture dans ta mémoire, ton identité ou ta configuration.** Tu lis, tu signales.
- ❌ **Aucune écriture avant validation du rapport.** Y compris une « petite correction évidente ».
- ❌ **Aucun `FAUX` sans preuve relançable.** Sans preuve, c'est `NON VÉRIFIABLE`, et ça ne bouge pas.
- ❌ **Aucune invention.** Tu déplaces, tu supprimes, tu corriges une valeur fausse par la valeur
  vraie. Tu ne reformules pas pour faire joli, tu ne complètes pas une règle incomplète.
- ❌ **Aucun travail sur un arbre git sale**, ni sur un projet non versionné sans accord explicite.
- ❌ **Aucune contradiction tranchée sans l'opérateur.**
- ❌ **Aucun lien laissé cassé** après un déplacement.
- ❌ **Aucune réécriture du corps d'un fichier d'agent.** Le frontmatter se corrige, le prompt non.
- ❌ **Aucune suppression de skill décidée par toi.**
- ❌ **Aucun texte qui sent le généré.**

---

## Références, à charger seulement au besoin

| Fichier | Quand le lire |
|---|---|
| `references/les-quatre-etages.md` | phases 3 et 5, choisir où sort un bloc. La fiche la plus utile du lot |
| `references/criteres-classement.md` | phase 3, pour trancher entre `SORTIR` et `ARCHIVE`, ou sur un cas limite |
| `references/anti-patterns.md` | phase 3, pour justifier un `SUPPRIMER` autrement qu'à l'instinct |
| `references/modeles.md` | phase 5, gabarit du fichier racine et gabarit d'une fiche |
| `references/fichiers-agents.md` | phase 2, quand le dépôt déclare des sous-agents |
| `references/memoire-de-l-agent.md` | quand une ligne de ta mémoire touche au sujet audité |
| `scripts/verif-refs.sh` | phase 2 et phase 5, chemins, commandes, globs et imports d'un `.md` |
| `scripts/verif-agents.sh` | phase 2, frontmatter des fichiers d'agents |
