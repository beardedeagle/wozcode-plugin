---
name: woz-settings
description: Manage WOZCODE plugin settings - toggle attribution, status line, spinner verbs.
allowed-tools: Bash(node *)
---

# WOZCODE Settings

Manage WOZCODE plugin settings. Harness-specific knobs (attribution, status line, spinner verbs, the live-reviewer toggles, …) live in `~/.claude/settings.json` under the `wozcode` key; the overarching KnowledgeBase backend (`knowledgeBaseProvider`, `knowledgeBaseServerUrl`) lives in `~/.woz/settings.json`. Prefer the `--show` / `--set` helper below over hand-editing — it routes each setting to the right store and applies side effects.

TRIGGER when: user says "woz settings", "woz config", "configure woz", "toggle attribution", "turn off status line", "disable co-authored-by", or similar.

## Usage

Run the settings helper to show or update settings:

### Show current settings
```bash
node --no-warnings=ExperimentalWarning ${CLAUDE_PLUGIN_ROOT}/scripts/settings-helper.js --show
```

Display the JSON output as a readable table for the user.

### Update a setting
```bash
node --no-warnings=ExperimentalWarning ${CLAUDE_PLUGIN_ROOT}/scripts/settings-helper.js --set <key> <value>
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
| `recall` | `true` | Session recall: the `Recall` MCP tool, the `/woz-recall` skill, and the background session indexer. Takes effect immediately. |
| `liveReviewer` | `true` | Live PostToolUse reviewer (Sonnet on every Edit) |
| `liveReviewerModel` | `claude-sonnet-4-6` | Model for the live pass. Unknown ids fall back to default. |
| `deepEditCountReviewer` | `true` | Every-N-edits deep-pass cadence trigger |
| `deepEditCountInterval` | `50` | Edits between deep cadence triggers (clamped to [5, 1000]) |
| `wozReviewModel` | `claude-opus-4-7` | Default model for `/woz-review` and the every-N-edits cadence. Accepts `provider/model` syntax (e.g. `azure-foundry/gpt-5.5-1`, `openai/gpt-5.5`) to auto-route through the WOZCODE router — requires `wozcode router start`. |
| `reviewerBaseUrl` | `''` | Optional ANTHROPIC_BASE_URL applied to both live and deep reviewers. Use this only when the router runs on a non-default host/port; routed model syntax above is the normal path. |
| `userEnabled` | `true` | Master plugin on/off. When `false`, pins `settings.agent` to `woz:code-free` (native Claude tools available, WOZCODE MCP disallowed). Same toggle as the desktop tray's "WOZCODE plugin: ON/OFF". |
| `showInMenuBar` | `true` | Whether the macOS menu-bar tray launches at login. Setting to `true` from the CLI re-launches the tray immediately. Setting to `false` unregisters the LaunchAgent; the running tray keeps going until quit. |

> **Build-dependent:** the live-reviewer knobs (`liveReviewer`, `liveReviewerModel`, `deepEditCountReviewer`, `deepEditCountInterval`, `wozReviewModel`) only apply on builds that bundle the reviewer surface. On other builds `--show` omits them and `--set` rejects them — treat the `--show` output as authoritative and don't offer to set a key it doesn't list.
>
> The KnowledgeBase backend settings (`knowledgeBaseProvider`, `knowledgeBaseServerUrl`) are independent: they live in `~/.woz/settings.json` and are **always** configurable, regardless of the build.

### About `alwaysLoadTools`

Claude Code can either load an MCP server's tool schemas into every session up-front, or defer them — in which case the model has to call the built-in `ToolSearch` tool once before it can use them.

- **`true` (default):** WOZCODE's tools (Search, Edit, Sql, Recall, Bash) are available immediately on every session. Best UX — the model uses them on the first turn without an extra discovery step.
- **`false`:** Tool schemas are deferred. Saves a small amount of system-prompt tokens per session, useful if you start lots of short sessions where you don't end up using WOZCODE's tools. The model will call `ToolSearch` to load them on first use.

Only affects WOZCODE's MCP server (`code`). Other MCP servers in your config are not touched.

Changes to this setting take effect on the **next Claude Code launch** because `.mcp.json` is read at startup, before session hooks run.

**Examples:**
```bash
# Disable attribution
node --no-warnings=ExperimentalWarning ${CLAUDE_PLUGIN_ROOT}/scripts/settings-helper.js --set attribution false

# Turn off status line tips
node --no-warnings=ExperimentalWarning ${CLAUDE_PLUGIN_ROOT}/scripts/settings-helper.js --set statusLineTips false

# Disable spinner verbs
node --no-warnings=ExperimentalWarning ${CLAUDE_PLUGIN_ROOT}/scripts/settings-helper.js --set spinnerVerbs false

# Defer WOZCODE tools behind ToolSearch (requires restart)
node --no-warnings=ExperimentalWarning ${CLAUDE_PLUGIN_ROOT}/scripts/settings-helper.js --set alwaysLoadTools false
```

After updating settings, tell the user:
- Most changes take effect immediately
- For `statusLine`, `attribution`, and `spinnerVerbs`: also tell them to run `/reload-plugins` so Claude Code picks up the change in the current session
- For `alwaysLoadTools`: tell them to **restart Claude Code** for the change to take effect (the helper already prints this reminder)
- For `recall`: takes effect immediately; the first Recall after enabling kicks off background indexing (no restart needed)
