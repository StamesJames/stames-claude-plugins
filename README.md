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

## Update

```sh
claude plugin marketplace update stames-plugins   # fetch the latest marketplace catalog
claude plugin update dev-workflow@stames-plugins  # update each installed plugin
claude plugin update agent-context@stames-plugins
```

Refresh the marketplace first, because `plugin update` only sees versions the local catalog
knows about. A plugin only updates when its `version` in `plugin.json` has been bumped.
Restart Claude Code afterwards to load the new version. If you installed a plugin with
`--scope project` or `--scope local`, pass the same `--scope` to `plugin update`.

## Adding a plugin

1. Create `plugins/<name>/` with `.claude-plugin/plugin.json` (same `name`) and its components.
2. Add an entry to `.claude-plugin/marketplace.json` with `"source": "./plugins/<name>"`.
3. Add a row to the table above.
4. Run `claude plugin validate .` and `claude plugin validate plugins/<name>`.

Bump `version` in a plugin's `plugin.json` whenever you change it, so installs pick up the update.
