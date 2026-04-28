---
name: woz-settings
description: Manage WOZCODE plugin settings - toggle attribution, status line, spinner verbs.
allowed-tools: Bash(node *)
---

# WOZCODE Settings

Manage WOZCODE plugin settings stored in `~/.claude/settings.json` under the `wozcode` key.

TRIGGER when: user says "woz settings", "woz config", "configure woz", "toggle attribution", "turn off status line", "disable co-authored-by", or similar.

## Usage

Run the settings helper to show or update settings:

### Show current settings
```bash
node ${CLAUDE_PLUGIN_ROOT}/scripts/settings-helper.js --show
```

Display the JSON output as a readable table for the user.

### Update a setting
```bash
node ${CLAUDE_PLUGIN_ROOT}/scripts/settings-helper.js --set <key> <value>
```

Where `<key>` is a setting name and `<value>` is `true` or `false`.

**Available settings:**

| Key | Default | Description |
|-----|---------|-------------|
| `attribution` | `true` | Co-Authored-By on commits + PR badge |
| `statusLine` | `true` | Master toggle for the WOZCODE status line |
| `statusLineSession` | `true` | Show session savings in status line |
| `statusLineLifetime` | `true` | Show lifetime savings in status line |
| `statusLineTips` | `true` | Show quick tips in status line |
| `statusLineShare` | `true` | Show /woz-share referral hint in status line |
| `spinnerVerbs` | `true` | WOZ-themed spinner verbs |
| `alwaysLoadTools` | `true` | Load WOZCODE MCP tools up-front instead of deferring them behind ToolSearch |

### About `alwaysLoadTools`

Claude Code can either load an MCP server's tool schemas into every session up-front, or defer them — in which case the model has to call the built-in `ToolSearch` tool once before it can use them.

- **`true` (default):** WOZCODE's tools (Search, Edit, Sql, Recall, Bash) are available immediately on every session. Best UX — the model uses them on the first turn without an extra discovery step.
- **`false`:** Tool schemas are deferred. Saves a small amount of system-prompt tokens per session, useful if you start lots of short sessions where you don't end up using WOZCODE's tools. The model will call `ToolSearch` to load them on first use.

Only affects WOZCODE's MCP server (`code`). Other MCP servers in your config are not touched.

Changes to this setting take effect on the **next Claude Code launch** because `.mcp.json` is read at startup, before session hooks run.

**Examples:**
```bash
# Disable attribution
node ${CLAUDE_PLUGIN_ROOT}/scripts/settings-helper.js --set attribution false

# Turn off status line tips
node ${CLAUDE_PLUGIN_ROOT}/scripts/settings-helper.js --set statusLineTips false

# Disable spinner verbs
node ${CLAUDE_PLUGIN_ROOT}/scripts/settings-helper.js --set spinnerVerbs false

# Defer WOZCODE tools behind ToolSearch (requires restart)
node ${CLAUDE_PLUGIN_ROOT}/scripts/settings-helper.js --set alwaysLoadTools false
```

After updating settings, tell the user:
- Most changes take effect immediately
- For `statusLine`, `attribution`, and `spinnerVerbs`: also tell them to run `/reload-plugins` so Claude Code picks up the change in the current session
- For `alwaysLoadTools`: tell them to **restart Claude Code** for the change to take effect (the helper already prints this reminder)
