---
name: write-plugins
description: Guide for creating Claude Code plugins. Use when writing, scaffolding, or structuring a plugin, configuring plugin.json, adding hooks/MCP servers/LSP servers/agents/skills to a plugin, debugging plugin issues, or understanding plugin architecture.
---

Copied from https://code.claude.com/docs/en/plugins-reference on 2026-02-28.

# Plugins reference

> Complete technical reference for Claude Code plugin system, including schemas, CLI commands, and component specifications.

This reference provides complete technical specifications for the Claude Code plugin system, including component schemas, CLI commands, and development tools.

A **plugin** is a self-contained directory of components that extends Claude Code with custom functionality. Plugin components include skills, agents, hooks, MCP servers, and LSP servers.

## Plugin components reference

### Skills

Plugins add skills to Claude Code, creating `/name` shortcuts that you or Claude can invoke.

**Location**: `skills/` or `commands/` directory in plugin root

**File format**: Skills are directories with `SKILL.md`; commands are simple markdown files

**Skill structure**:

```text
skills/
├── pdf-processor/
│   ├── SKILL.md
│   ├── reference.md (optional)
│   └── scripts/ (optional)
└── code-reviewer/
    └── SKILL.md
```

**Integration behavior**:

* Skills and commands are automatically discovered when the plugin is installed
* Claude can invoke them automatically based on task context
* Skills can include supporting files alongside SKILL.md

For complete details, see [Skills](https://code.claude.com/docs/en/skills).

### Agents

Plugins can provide specialized subagents for specific tasks that Claude can invoke automatically when appropriate.

**Location**: `agents/` directory in plugin root

**File format**: Markdown files describing agent capabilities

**Agent structure**:

```markdown
---
name: agent-name
description: What this agent specializes in and when Claude should invoke it
---

Detailed system prompt for the agent describing its role, expertise, and behavior.
```

**Integration points**:

* Agents appear in the `/agents` interface
* Claude can invoke agents automatically based on task context
* Agents can be invoked manually by users
* Plugin agents work alongside built-in Claude agents

For complete details, see [Subagents](https://code.claude.com/docs/en/sub-agents).

### Hooks

Plugins can provide event handlers that respond to Claude Code events automatically.

**Location**: `hooks/hooks.json` in plugin root, or inline in plugin.json

**Format**: JSON configuration with event matchers and actions

**Hook configuration**:

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Write|Edit",
        "hooks": [
          {
            "type": "command",
            "command": "${CLAUDE_PLUGIN_ROOT}/scripts/format-code.sh"
          }
        ]
      }
    ]
  }
}
```

**Available events**:

* `PreToolUse`: Before Claude uses any tool
* `PostToolUse`: After Claude successfully uses any tool
* `PostToolUseFailure`: After Claude tool execution fails
* `PermissionRequest`: When a permission dialog is shown
* `UserPromptSubmit`: When user submits a prompt
* `Notification`: When Claude Code sends notifications
* `Stop`: When Claude attempts to stop
* `SubagentStart`: When a subagent is started
* `SubagentStop`: When a subagent attempts to stop
* `SessionStart`: At the beginning of sessions
* `SessionEnd`: At the end of sessions
* `TeammateIdle`: When an agent team teammate is about to go idle
* `TaskCompleted`: When a task is being marked as completed
* `PreCompact`: Before conversation history is compacted

**Hook types**:

* `command`: Execute shell commands or scripts
* `prompt`: Evaluate a prompt with an LLM (uses `$ARGUMENTS` placeholder for context)
* `agent`: Run an agentic verifier with tools for complex verification tasks

### MCP servers

Plugins can bundle Model Context Protocol (MCP) servers to connect Claude Code with external tools and services.

**Location**: `.mcp.json` in plugin root, or inline in plugin.json

**Format**: Standard MCP server configuration

**MCP server configuration**:

```json
{
  "mcpServers": {
    "plugin-database": {
      "command": "${CLAUDE_PLUGIN_ROOT}/servers/db-server",
      "args": ["--config", "${CLAUDE_PLUGIN_ROOT}/config.json"],
      "env": {
        "DB_PATH": "${CLAUDE_PLUGIN_ROOT}/data"
      }
    },
    "plugin-api-client": {
      "command": "npx",
      "args": ["@company/mcp-server", "--plugin-mode"],
      "cwd": "${CLAUDE_PLUGIN_ROOT}"
    }
  }
}
```

**Integration behavior**:

* Plugin MCP servers start automatically when the plugin is enabled
* Servers appear as standard MCP tools in Claude's toolkit
* Server capabilities integrate seamlessly with Claude's existing tools
* Plugin servers can be configured independently of user MCP servers

### LSP servers

Plugins can provide [Language Server Protocol](https://microsoft.github.io/language-server-protocol/) (LSP) servers to give Claude real-time code intelligence while working on your codebase.

LSP integration provides:

* **Instant diagnostics**: Claude sees errors and warnings immediately after each edit
* **Code navigation**: go to definition, find references, and hover information
* **Language awareness**: type information and documentation for code symbols

**Location**: `.lsp.json` in plugin root, or inline in `plugin.json`

**Format**: JSON configuration mapping language server names to their configurations

**`.lsp.json` file format**:

```json
{
  "go": {
    "command": "gopls",
    "args": ["serve"],
    "extensionToLanguage": {
      ".go": "go"
    }
  }
}
```

**Inline in `plugin.json`**:

```json
{
  "name": "my-plugin",
  "lspServers": {
    "go": {
      "command": "gopls",
      "args": ["serve"],
      "extensionToLanguage": {
        ".go": "go"
      }
    }
  }
}
```

**Required fields:**

| Field                 | Description                                  |
| :-------------------- | :------------------------------------------- |
| `command`             | The LSP binary to execute (must be in PATH)  |
| `extensionToLanguage` | Maps file extensions to language identifiers |

**Optional fields:**

| Field                   | Description                                               |
| :---------------------- | :-------------------------------------------------------- |
| `args`                  | Command-line arguments for the LSP server                 |
| `transport`             | Communication transport: `stdio` (default) or `socket`    |
| `env`                   | Environment variables to set when starting the server     |
| `initializationOptions` | Options passed to the server during initialization        |
| `settings`              | Settings passed via `workspace/didChangeConfiguration`    |
| `workspaceFolder`       | Workspace folder path for the server                      |
| `startupTimeout`        | Max time to wait for server startup (milliseconds)        |
| `shutdownTimeout`       | Max time to wait for graceful shutdown (milliseconds)     |
| `restartOnCrash`        | Whether to automatically restart the server if it crashes |
| `maxRestarts`           | Maximum number of restart attempts before giving up       |

---

## Plugin installation scopes

When you install a plugin, you choose a **scope** that determines where the plugin is available and who else can use it:

| Scope     | Settings file                                   | Use case                                                 |
| :-------- | :---------------------------------------------- | :------------------------------------------------------- |
| `user`    | `~/.claude/settings.json`                       | Personal plugins available across all projects (default) |
| `project` | `.claude/settings.json`                         | Team plugins shared via version control                  |
| `local`   | `.claude/settings.local.json`                   | Project-specific plugins, gitignored                     |
| `managed` | [Managed settings](https://code.claude.com/docs/en/settings#settings-files) | Managed plugins (read-only, update only)                 |

Plugins use the same scope system as other Claude Code configurations. For installation instructions and scope flags, see [Install plugins](https://code.claude.com/docs/en/discover-plugins#install-plugins). For a complete explanation of scopes, see [Configuration scopes](https://code.claude.com/docs/en/settings#configuration-scopes).

---

## Plugin manifest schema

For the complete manifest schema including all required fields, metadata fields, component path fields, path behavior rules, and environment variables, see [references/full-spec.md](references/full-spec.md#plugin-manifest-schema).

---

## Plugin directory structure

### Standard plugin layout

A complete plugin follows this structure:

```text
enterprise-plugin/
├── .claude-plugin/           # Metadata directory (optional)
│   └── plugin.json             # plugin manifest
├── commands/                 # Default command location
│   ├── status.md
│   └── logs.md
├── agents/                   # Default agent location
│   ├── security-reviewer.md
│   ├── performance-tester.md
│   └── compliance-checker.md
├── skills/                   # Agent Skills
│   ├── code-reviewer/
│   │   └── SKILL.md
│   └── pdf-processor/
│       ├── SKILL.md
│       └── scripts/
├── hooks/                    # Hook configurations
│   ├── hooks.json           # Main hook config
│   └── security-hooks.json  # Additional hooks
├── settings.json            # Default settings for the plugin
├── .mcp.json                # MCP server definitions
├── .lsp.json                # LSP server configurations
├── scripts/                 # Hook and utility scripts
│   ├── security-scan.sh
│   ├── format-code.py
│   └── deploy.js
├── LICENSE                  # License file
└── CHANGELOG.md             # Version history
```

The `.claude-plugin/` directory contains the `plugin.json` file. All other directories (commands/, agents/, skills/, hooks/) must be at the plugin root, not inside `.claude-plugin/`.

### File locations reference

| Component       | Default Location             | Purpose                                                                                                                   |
| :-------------- | :--------------------------- | :------------------------------------------------------------------------------------------------------------------------ |
| **Manifest**    | `.claude-plugin/plugin.json` | Plugin metadata and configuration (optional)                                                                              |
| **Commands**    | `commands/`                  | Skill Markdown files (legacy; use `skills/` for new skills)                                                               |
| **Agents**      | `agents/`                    | Subagent Markdown files                                                                                                   |
| **Skills**      | `skills/`                    | Skills with `<name>/SKILL.md` structure                                                                                   |
| **Hooks**       | `hooks/hooks.json`           | Hook configuration                                                                                                        |
| **MCP servers** | `.mcp.json`                  | MCP server definitions                                                                                                    |
| **LSP servers** | `.lsp.json`                  | Language server configurations                                                                                            |
| **Settings**    | `settings.json`              | Default configuration applied when the plugin is enabled. Only [agent](https://code.claude.com/docs/en/sub-agents) settings are currently supported |

---

## CLI commands reference

For complete CLI command details including all options, arguments, and examples for `plugin install`, `plugin uninstall`, `plugin enable`, `plugin disable`, and `plugin update`, see [references/full-spec.md](references/full-spec.md#cli-commands-reference).

---

## Debugging and development tools

### Debugging commands

Use `claude --debug` (or `/debug` within the TUI) to see plugin loading details:

This shows:

* Which plugins are being loaded
* Any errors in plugin manifests
* Command, agent, and hook registration
* MCP server initialization

### Common issues

| Issue                               | Cause                           | Solution                                                                          |
| :---------------------------------- | :------------------------------ | :-------------------------------------------------------------------------------- |
| Plugin not loading                  | Invalid `plugin.json`           | Validate JSON syntax with `claude plugin validate` or `/plugin validate`          |
| Commands not appearing              | Wrong directory structure       | Ensure `commands/` at root, not in `.claude-plugin/`                              |
| Hooks not firing                    | Script not executable           | Run `chmod +x script.sh`                                                          |
| MCP server fails                    | Missing `${CLAUDE_PLUGIN_ROOT}` | Use variable for all plugin paths                                                 |
| Path errors                         | Absolute paths used             | All paths must be relative and start with `./`                                    |
| LSP `Executable not found in $PATH` | Language server not installed   | Install the binary (e.g., `npm install -g typescript-language-server typescript`) |

### Example error messages

**Manifest validation errors**:

* `Invalid JSON syntax: Unexpected token } in JSON at position 142`: check for missing commas, extra commas, or unquoted strings
* `Plugin has an invalid manifest file at .claude-plugin/plugin.json. Validation errors: name: Required`: a required field is missing
* `Plugin has a corrupt manifest file at .claude-plugin/plugin.json. JSON parse error: ...`: JSON syntax error

**Plugin loading errors**:

* `Warning: No commands found in plugin my-plugin custom directory: ./cmds. Expected .md files or SKILL.md in subdirectories.`: command path exists but contains no valid command files
* `Plugin directory not found at path: ./plugins/my-plugin. Check that the marketplace entry has the correct path.`: the `source` path in marketplace.json points to a non-existent directory
* `Plugin my-plugin has conflicting manifests: both plugin.json and marketplace entry specify components.`: remove duplicate component definitions or remove `strict: false` in marketplace entry

### Hook troubleshooting

**Hook script not executing**:

1. Check the script is executable: `chmod +x ./scripts/your-script.sh`
2. Verify the shebang line: First line should be `#!/bin/bash` or `#!/usr/bin/env bash`
3. Check the path uses `${CLAUDE_PLUGIN_ROOT}`: `"command": "${CLAUDE_PLUGIN_ROOT}/scripts/your-script.sh"`
4. Test the script manually: `./scripts/your-script.sh`

**Hook not triggering on expected events**:

1. Verify the event name is correct (case-sensitive): `PostToolUse`, not `postToolUse`
2. Check the matcher pattern matches your tools: `"matcher": "Write|Edit"` for file operations
3. Confirm the hook type is valid: `command`, `prompt`, or `agent`

### MCP server troubleshooting

**Server not starting**:

1. Check the command exists and is executable
2. Verify all paths use `${CLAUDE_PLUGIN_ROOT}` variable
3. Check the MCP server logs: `claude --debug` shows initialization errors
4. Test the server manually outside of Claude Code

**Server tools not appearing**:

1. Ensure the server is properly configured in `.mcp.json` or `plugin.json`
2. Verify the server implements the MCP protocol correctly
3. Check for connection timeouts in debug output

### Directory structure mistakes

**Symptoms**: Plugin loads but components (commands, agents, hooks) are missing.

**Correct structure**: Components must be at the plugin root, not inside `.claude-plugin/`. Only `plugin.json` belongs in `.claude-plugin/`.

```text
my-plugin/
├── .claude-plugin/
│   └── plugin.json      ← Only manifest here
├── commands/            ← At root level
├── agents/              ← At root level
└── hooks/               ← At root level
```

If your components are inside `.claude-plugin/`, move them to the plugin root.

**Debug checklist**:

1. Run `claude --debug` and look for "loading plugin" messages
2. Check that each component directory is listed in the debug output
3. Verify file permissions allow reading the plugin files

---

## Distribution and versioning reference

For complete versioning guidance including semantic versioning rules, best practices, and caching behavior, see [references/full-spec.md](references/full-spec.md#distribution-and-versioning-reference).

---

## Plugin caching and file resolution

Plugins are specified in one of two ways:

* Through `claude --plugin-dir`, for the duration of a session.
* Through a marketplace, installed for future sessions.

For security and verification purposes, Claude Code copies *marketplace* plugins to the user's local **plugin cache** (`~/.claude/plugins/cache`) rather than using them in-place. Understanding this behavior is important when developing plugins that reference external files.

### Path traversal limitations

Installed plugins cannot reference files outside their directory. Paths that traverse outside the plugin root (such as `../shared-utils`) will not work after installation because those external files are not copied to the cache.

### Working with external dependencies

If your plugin needs to access files outside its directory, you can create symbolic links to external files within your plugin directory. Symlinks are honored during the copy process:

```bash
# Inside your plugin directory
ln -s /path/to/shared-utils ./shared-utils
```

The symlinked content will be copied into the plugin cache. This provides flexibility while maintaining the security benefits of the caching system.

---

## See also

* [Plugins](https://code.claude.com/docs/en/plugins) - Tutorials and practical usage
* [Plugin marketplaces](https://code.claude.com/docs/en/plugin-marketplaces) - Creating and managing marketplaces
* [Skills](https://code.claude.com/docs/en/skills) - Skill development details
* [Subagents](https://code.claude.com/docs/en/sub-agents) - Agent configuration and capabilities
* [Hooks](https://code.claude.com/docs/en/hooks) - Event handling and automation
* [MCP](https://code.claude.com/docs/en/mcp) - External tool integration
* [Settings](https://code.claude.com/docs/en/settings) - Configuration options for plugins
