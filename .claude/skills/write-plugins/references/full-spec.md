# Plugin manifest schema

The `.claude-plugin/plugin.json` file defines your plugin's metadata and configuration. This section documents all supported fields and options.

The manifest is optional. If omitted, Claude Code auto-discovers components in [default locations](https://code.claude.com/docs/en/plugins-reference#file-locations-reference) and derives the plugin name from the directory name. Use a manifest when you need to provide metadata or custom component paths.

### Complete schema

```json
{
  "name": "plugin-name",
  "version": "1.2.0",
  "description": "Brief plugin description",
  "author": {
    "name": "Author Name",
    "email": "author@example.com",
    "url": "https://github.com/author"
  },
  "homepage": "https://docs.example.com/plugin",
  "repository": "https://github.com/author/plugin",
  "license": "MIT",
  "keywords": ["keyword1", "keyword2"],
  "commands": ["./custom/commands/special.md"],
  "agents": "./custom/agents/",
  "skills": "./custom/skills/",
  "hooks": "./config/hooks.json",
  "mcpServers": "./mcp-config.json",
  "outputStyles": "./styles/",
  "lspServers": "./.lsp.json"
}
```

### Required fields

If you include a manifest, `name` is the only required field.

| Field  | Type   | Description                               | Example              |
| :----- | :----- | :---------------------------------------- | :------------------- |
| `name` | string | Unique identifier (kebab-case, no spaces) | `"deployment-tools"` |

This name is used for namespacing components. For example, in the UI, the
agent `agent-creator` for the plugin with name `plugin-dev` will appear as
`plugin-dev:agent-creator`.

### Metadata fields

| Field         | Type   | Description                                                                                                                 | Example                                            |
| :------------ | :----- | :-------------------------------------------------------------------------------------------------------------------------- | :------------------------------------------------- |
| `version`     | string | Semantic version. If also set in the marketplace entry, `plugin.json` takes priority. You only need to set it in one place. | `"2.1.0"`                                          |
| `description` | string | Brief explanation of plugin purpose                                                                                         | `"Deployment automation tools"`                    |
| `author`      | object | Author information                                                                                                          | `{"name": "Dev Team", "email": "dev@company.com"}` |
| `homepage`    | string | Documentation URL                                                                                                           | `"https://docs.example.com"`                       |
| `repository`  | string | Source code URL                                                                                                             | `"https://github.com/user/plugin"`                 |
| `license`     | string | License identifier                                                                                                          | `"MIT"`, `"Apache-2.0"`                            |
| `keywords`    | array  | Discovery tags                                                                                                              | `["deployment", "ci-cd"]`                          |

### Component path fields

| Field          | Type                  | Description                                                                                                                                               | Example                                |
| :------------- | :-------------------- | :-------------------------------------------------------------------------------------------------------------------------------------------------------- | :------------------------------------- |
| `commands`     | string\|array         | Additional command files/directories                                                                                                                      | `"./custom/cmd.md"` or `["./cmd1.md"]` |
| `agents`       | string\|array         | Additional agent files                                                                                                                                    | `"./custom/agents/reviewer.md"`        |
| `skills`       | string\|array         | Additional skill directories                                                                                                                              | `"./custom/skills/"`                   |
| `hooks`        | string\|array\|object | Hook config paths or inline config                                                                                                                        | `"./my-extra-hooks.json"`              |
| `mcpServers`   | string\|array\|object | MCP config paths or inline config                                                                                                                         | `"./my-extra-mcp-config.json"`         |
| `outputStyles` | string\|array         | Additional output style files/directories                                                                                                                 | `"./styles/"`                          |
| `lspServers`   | string\|array\|object | [Language Server Protocol](https://microsoft.github.io/language-server-protocol/) configs for code intelligence (go to definition, find references, etc.) | `"./.lsp.json"`                        |

### Path behavior rules

**Important**: Custom paths supplement default directories - they don't replace them.

* If `commands/` exists, it's loaded in addition to custom command paths
* All paths must be relative to plugin root and start with `./`
* Commands from custom paths use the same naming and namespacing rules
* Multiple paths can be specified as arrays for flexibility

**Path examples**:

```json
{
  "commands": [
    "./specialized/deploy.md",
    "./utilities/batch-process.md"
  ],
  "agents": [
    "./custom-agents/reviewer.md",
    "./custom-agents/tester.md"
  ]
}
```

### Environment variables

**`${CLAUDE_PLUGIN_ROOT}`**: Contains the absolute path to your plugin directory. Use this in hooks, MCP servers, and scripts to ensure correct paths regardless of installation location.

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "${CLAUDE_PLUGIN_ROOT}/scripts/process.sh"
          }
        ]
      }
    ]
  }
}
```

---

# CLI commands reference

Claude Code provides CLI commands for non-interactive plugin management, useful for scripting and automation.

### plugin install

Install a plugin from available marketplaces.

```bash
claude plugin install <plugin> [options]
```

**Arguments:**

* `<plugin>`: Plugin name or `plugin-name@marketplace-name` for a specific marketplace

**Options:**

| Option                | Description                                       | Default |
| :-------------------- | :------------------------------------------------ | :------ |
| `-s, --scope <scope>` | Installation scope: `user`, `project`, or `local` | `user`  |
| `-h, --help`          | Display help for command                          |         |

Scope determines which settings file the installed plugin is added to. For example, --scope project writes to `enabledPlugins` in .claude/settings.json, making the plugin available to everyone who clones the project repository.

**Examples:**

```bash
# Install to user scope (default)
claude plugin install formatter@my-marketplace

# Install to project scope (shared with team)
claude plugin install formatter@my-marketplace --scope project

# Install to local scope (gitignored)
claude plugin install formatter@my-marketplace --scope local
```

### plugin uninstall

Remove an installed plugin.

```bash
claude plugin uninstall <plugin> [options]
```

**Arguments:**

* `<plugin>`: Plugin name or `plugin-name@marketplace-name`

**Options:**

| Option                | Description                                         | Default |
| :-------------------- | :-------------------------------------------------- | :------ |
| `-s, --scope <scope>` | Uninstall from scope: `user`, `project`, or `local` | `user`  |
| `-h, --help`          | Display help for command                            |         |

**Aliases:** `remove`, `rm`

### plugin enable

Enable a disabled plugin.

```bash
claude plugin enable <plugin> [options]
```

**Arguments:**

* `<plugin>`: Plugin name or `plugin-name@marketplace-name`

**Options:**

| Option                | Description                                    | Default |
| :-------------------- | :--------------------------------------------- | :------ |
| `-s, --scope <scope>` | Scope to enable: `user`, `project`, or `local` | `user`  |
| `-h, --help`          | Display help for command                       |         |

### plugin disable

Disable a plugin without uninstalling it.

```bash
claude plugin disable <plugin> [options]
```

**Arguments:**

* `<plugin>`: Plugin name or `plugin-name@marketplace-name`

**Options:**

| Option                | Description                                     | Default |
| :-------------------- | :---------------------------------------------- | :------ |
| `-s, --scope <scope>` | Scope to disable: `user`, `project`, or `local` | `user`  |
| `-h, --help`          | Display help for command                        |         |

### plugin update

Update a plugin to the latest version.

```bash
claude plugin update <plugin> [options]
```

**Arguments:**

* `<plugin>`: Plugin name or `plugin-name@marketplace-name`

**Options:**

| Option                | Description                                               | Default |
| :-------------------- | :-------------------------------------------------------- | :------ |
| `-s, --scope <scope>` | Scope to update: `user`, `project`, `local`, or `managed` | `user`  |
| `-h, --help`          | Display help for command                                  |         |

---

# Distribution and versioning reference

### Version management

Follow semantic versioning for plugin releases:

```json
{
  "name": "my-plugin",
  "version": "2.1.0"
}
```

**Version format**: `MAJOR.MINOR.PATCH`

* **MAJOR**: Breaking changes (incompatible API changes)
* **MINOR**: New features (backward-compatible additions)
* **PATCH**: Bug fixes (backward-compatible fixes)

**Best practices**:

* Start at `1.0.0` for your first stable release
* Update the version in `plugin.json` before distributing changes
* Document changes in a `CHANGELOG.md` file
* Use pre-release versions like `2.0.0-beta.1` for testing

Claude Code uses the version to determine whether to update your plugin. If you change your plugin's code but don't bump the version in `plugin.json`, your plugin's existing users won't see your changes due to caching.

If your plugin is within a [marketplace](https://code.claude.com/docs/en/plugin-marketplaces) directory, you can manage the version through `marketplace.json` instead and omit the `version` field from `plugin.json`.
