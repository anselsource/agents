# Known Issues

External tooling bugs and workarounds encountered during development.

## 2026-02-26

**Cowork ignores `marketplace.metadata.pluginRoot`**
Claude 1.1.4498 (24f768). Plugin installs break when `pluginRoot` is set and `source` uses a path relative to it.
Workaround: use full relative paths in `source` (e.g., `./plugins/forecast`).
