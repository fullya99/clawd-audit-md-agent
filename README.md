# clawd-audit-md-agent est remplacé par clawd-stack-agent

Ce dépôt n'est plus maintenu. Le skill `clawd-audit-md-agent` est absorbé par `clawd-stack-agent`, qui reprend son
contenu avec le reste de la convention de contexte.

## Passer à clawd-stack-agent

L'installeur de Hermes a refusé l'ancien paquet le 2026-08-13, son scanner s'arrête sur la
mention de `CLAUDE.md`, et le nouveau porte la même signature. On l'installe donc par un clone,
qui se met à jour par un `git pull`.

Sur Hermes Agent :

```bash
git clone https://github.com/fullya99/clawd-stack-agent.git \
  ~/.hermes/skills/housekeeping/clawd-stack-agent
```

Sur OpenClaw :

```bash
git clone https://github.com/fullya99/clawd-stack-agent.git \
  ~/.openclaw/workspace/skills/clawd-stack-agent
```

Sur Codex :

```bash
git clone https://github.com/fullya99/clawd-stack-agent.git \
  .agents/skills/clawd-stack-agent
```

Pour tous tes dépôts, clone dans `~/.agents/skills/clawd-stack-agent` à la place. Le README de
`github.com/fullya99/clawd-stack-agent` donne le détail.

## Les anciennes versions

Les tags et les releases de ce dépôt restent disponibles, archives `.zip` comprises.
