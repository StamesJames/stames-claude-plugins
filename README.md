# stames-plugins

Personal Claude Code plugin marketplace.

| Plugin | Description |
| --- | --- |
| [dev-workflow](plugins/dev-workflow) | Plan-first dev workflow: explore and plan features, analyse code quality, then implement or fix list items unattended with one fresh agent per item. |
| [agent-context](plugins/agent-context) | Keep CLAUDE.md lean: tidy it under 200 lines by moving detail into path-scoped rules or on-demand `CLAUDE_<TOPIC>.md` files listed in a Context files index. |

## Install

```sh
claude plugin marketplace add StamesJames/stames-claude-plugins
claude plugin install dev-workflow@stames-plugins
claude plugin install agent-context@stames-plugins
```

Update later with `claude plugin marketplace update stames-plugins`.

## Adding a plugin

1. Create `plugins/<name>/` with `.claude-plugin/plugin.json` (same `name`) and its components.
2. Add an entry to `.claude-plugin/marketplace.json` with `"source": "./plugins/<name>"`.
3. Run `claude plugin validate .` and `claude plugin validate plugins/<name>`.

Bump `version` in a plugin's `plugin.json` whenever you change it, so installs pick up the update.
