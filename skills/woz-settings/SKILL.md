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

**Examples:**
```bash
# Disable attribution
node ${CLAUDE_PLUGIN_ROOT}/scripts/settings-helper.js --set attribution false

# Turn off status line tips
node ${CLAUDE_PLUGIN_ROOT}/scripts/settings-helper.js --set statusLineTips false

# Disable spinner verbs
node ${CLAUDE_PLUGIN_ROOT}/scripts/settings-helper.js --set spinnerVerbs false
```

After updating settings, tell the user:
- All changes take effect immediately
- For `statusLine`, `attribution`, and `spinnerVerbs`: also tell them to run `/reload-plugins` so Claude Code picks up the change in the current session
