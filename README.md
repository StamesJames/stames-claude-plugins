# stames-plugins

Personal Claude Code plugin marketplace.

| Plugin                               | Description                                                                                                                  |
| ------------------------------------ | ---------------------------------------------------------------------------------------------------------------------------- |
| [dev-workflow](plugins/dev-workflow) | Analysis/plan workflow: code analysis, fixing findings, planning, pitching and implementing features via running list files. |

## Install

```sh
claude plugin marketplace add StamesJames/stames-claude-plugins
claude plugin install dev-workflow@stames-plugins
```

Update later with `claude plugin marketplace update stames-plugins`.

## Adding a plugin

1. Create `plugins/<name>/` with `.claude-plugin/plugin.json` (same `name`) and its components.
2. Add an entry to `.claude-plugin/marketplace.json` with `"source": "./plugins/<name>"`.
3. Run `claude plugin validate .` and `claude plugin validate plugins/<name>`.

Bump `version` in a plugin's `plugin.json` whenever you change it, so installs pick up the update.
