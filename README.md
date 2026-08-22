# clawd-audit-md-agent

**Il raccourcit le `CLAUDE.md` ou l'`AGENTS.md` d'un projet.** Format agentskills.io. Il lit le
fichier, va chercher dans le code ce qui confirme ou contredit chaque ligne, et rend un plan : ce
qui reste à la racine, ce qui sort, où ça sort, ce qui s'archive, ce qui se supprime. Rien ne
s'écrit avant validation.

Le fichier racine est chargé en entier au début de chaque session, donc chaque ligne est payée pour
toujours, qu'elle serve ou non. Presque tout ce qu'il contient peut vivre dans un skill ou une
fiche liée et ne charger que quand ça sert.

Testé pour Hermes Agent et OpenClaw. Il marche aussi chez Claude Code et Codex, qui lisent le même
standard. La variante avec manifeste et slash command s'appelle `clawd-audit-md`.

## Ce qu'il fait, et que les autres ne font pas

**Il sait où sortir un bloc.** Un skill charge son corps seulement quand il sert, et sa description
reste visible, donc il peut se déclencher tout seul. Une fiche liée ne charge que si l'agent décide
d'ouvrir le lien, donc elle ne garantit rien. La règle qui tranche : une contrainte qui doit être
appliquée va en skill, la fiche porte ce qu'on consulte. Sur un dépôt lu par Claude Code, il
propose en plus les règles liées à des chemins, qui se chargent toutes seules.

**Et il ne coupe rien sans preuve.** Un chemin cité doit exister, une commande citée doit être dans
le `package.json` ou le `Makefile`, un glob de règle doit matcher au moins un fichier, un import
`@fichier` doit résoudre, une convention décrite se compte sur un échantillon. Une ligne contredite
est marquée fausse **avec sa preuve**. Une ligne qu'il n'arrive pas à trancher est marquée non
vérifiable et ne bouge pas, jamais fausse par défaut.

Il couvre aussi les fichiers de définition de sous-agents, qui cassent différemment : un
frontmatter mal formé n'est pas chargé, sans erreur et sans avertissement. Et il sait passer sur un
catalogue de skills pour y trouver les doublons et les périmés.

## Il ne touche pas à ta mémoire

`SOUL.md`, `USER.md`, `MEMORY.md`, le journal quotidien : il les lit, il signale une contradiction
avec le dépôt, il n'écrit jamais dedans. C'est une décision de conception, expliquée dans
`references/memoire-de-l-agent.md`. Un audit qui corrige sa propre mémoire n'a plus de juge, et
l'écriture en mémoire d'agent est le premier vecteur d'attaque documenté de cet écosystème.

## Installation

Chez Hermes, dans `~/.hermes/skills/`. Le dossier de catégorie est optionnel, le scan est récursif.

```bash
git clone https://github.com/fullya99/clawd-audit-md-agent ~/.hermes/skills/clawd-audit-md-agent
```

Chez OpenClaw, dans le workspace.

```bash
git clone https://github.com/fullya99/clawd-audit-md-agent ~/.openclaw/workspace/skills/clawd-audit-md-agent
```

Chez Codex ou Claude Code, à la portée voulue.

```bash
git clone https://github.com/fullya99/clawd-audit-md-agent .agents/skills/clawd-audit-md-agent
git clone https://github.com/fullya99/clawd-audit-md-agent ~/.claude/skills/clawd-audit-md-agent
```

### Pourquoi le clone et pas `hermes skills install`

L'installeur officiel d'Hermes le refusera, et ce n'est pas réparable de notre côté.

Son scanner porte une règle `agent_config_mod`, de sévérité critique, dont le motif est
littéralement `AGENTS\.md|CLAUDE\.md|\.cursorrules|\.clinerules`. Elle se déclenche sur la simple
**mention** de ces noms de fichiers, dans n'importe quel `.md` ou script du skill. Or ces noms sont
le sujet même de ce skill, il les cite des dizaines de fois.

Une trouvaille critique donne un verdict `dangerous`. Pour une source communautaire, `dangerous`
vaut blocage, et `--force` ne le lève pas : la condition est écrite en dur dans
`tools/skills_guard.py`, vérifiée sur `main` le 2026-08-23.

Le scanner a raison sur le fond, sa règle vise les skills qui persistent des instructions
malveillantes dans la config d'un agent. Il n'a simplement aucun moyen de distinguer « écrit dans
`CLAUDE.md` » de « parle de `CLAUDE.md` ». Contourner ça en déguisant les noms de fichiers serait
de l'évasion de scanner, donc on ne le fait pas. Le clone est le mode de distribution correct pour
cette classe d'outil : un skill qui lit tes fichiers d'instructions, tu l'installes en le lisant.

Chez OpenClaw et chez Claude Code, aucun scan de ce type, le clone suffit.

## Usage

Nomme le projet, puis demande.

```
audite les instructions de ~/projets/mon-appli
```

Il établit d'abord sur quel projet il travaille, vérifie que l'arbre git est propre, inventorie les
fichiers d'instructions, lance ses vérifications, pose les questions des contradictions trouvées,
et présente un rapport en quatre morceaux : le tableau des preuves, le diff du fichier racine, la
liste des fiches à créer avec leur taille, le tableau des suppressions. Tu approuves, tu corriges,
ou tu refuses bloc par bloc. Il écrit après.

Les deux contrôles s'utilisent aussi tout seuls :

```bash
bash scripts/verif-refs.sh --format=md CLAUDE.md AGENTS.md
bash scripts/verif-agents.sh .claude/agents/
```

Chacun sort 1 s'il reste une référence fausse, donc ils s'accrochent à un hook ou à une CI. Ils ne
dépendent que de `find`, `grep`, `sed` et `awk`, et n'accèdent pas au réseau.

## Ce qu'il ne fait pas

Il n'invente rien. Il déplace, il supprime, il corrige une valeur fausse par la valeur vraie. Il ne
reformule pas une règle pour la rendre plus jolie et il ne complète pas une règle incomplète.

Il n'écrit pas un fichier d'instructions de zéro. Un projet qui n'en a pas est hors de son
périmètre.

Il ne tranche jamais une contradiction tout seul, même quand une version semble évidente.

Il ne réécrit pas le corps d'un fichier d'agent, ni ne supprime un skill de son propre chef.

Il ne travaille pas sur un arbre git sale, et il demande une autorisation explicite sur un projet
non versionné.

## Licence

MIT.
