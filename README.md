# agent-skills

Personal Claude Code skills, packaged as a plugin marketplace.

## Install

```
/plugin marketplace add rizese/agent-skills
/plugin install dumbdown@rizese
```

## Plugins

- **dumbdown**: `/dumbdown` answers in as few plain sentences as possible until you say stop.

## Adding a plugin

1. Create `plugins/<name>/.claude-plugin/plugin.json`.
2. Put skills in `plugins/<name>/skills/<skill>/SKILL.md`.
3. Add an entry to `.claude-plugin/marketplace.json`.
