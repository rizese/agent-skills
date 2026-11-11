# agent-skills

Personal Claude Code skills, packaged as one plugin named `rizese`. Every skill shows up as `rizese:<skill>`.

## Install

```
/plugin marketplace add rizese/agent-skills
/plugin install rizese@rizese
```

## Skills

- **dumbdown**: `/rizese:dumbdown` answers in as few plain sentences as possible until you say stop.
- **socratic-review**: `/rizese:socratic-review` reviews PRs and posts inline comments written only as questions, so the author finds the problem themselves.

## Adding a skill

Put it in `skills/<skill>/SKILL.md` and push. No manifest change is needed: the plugin has no `version`, so each commit counts as a new version and `/plugin marketplace update rizese` (or auto-update) picks it up.
