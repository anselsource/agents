# Known Issues

External tooling bugs and workarounds encountered during development.

## 2026-02-26

**Cowork ignores `marketplace.metadata.pluginRoot`**
Claude 1.1.4498 (24f768). Plugin installs break when `pluginRoot` is set and `source` uses a path relative to it.
Workaround: use full relative paths in `source` (e.g., `./plugins/forecast`).

## 2026-02-28

**Cowork auto-generates `plugin.json` with invalid `source` field**
When a plugin has no `.claude-plugin/plugin.json`, Claude Code auto-generates one from the marketplace entry, leaking the `source` field (a marketplace-only field) into the plugin manifest. The plugin loader then silently rejects the plugin — it appears installed but the skill is `Unknown` at invocation time.
Workaround: add an explicit `.claude-plugin/plugin.json` to the plugin source with only valid manifest fields (`name`, `description`). Keep `version` in `marketplace.json` only per the docs recommendation for relative-path plugins.
