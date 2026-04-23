# WOZCODE Plugin for Claude Code

Smarter tools for Claude Code that reduce token usage and cost. Replaces built-in file tools with optimized alternatives — fewer tokens per tool call means cheaper sessions that compound over time.

## Getting Started

### 1. Install

From GitHub — inside a Claude Code session, run:

```
/plugin marketplace add WithWoz/wozcode-plugin
/plugin install woz@wozcode-marketplace
```

### 2. Restart Claude Code

Quit your current session and start a new one:

```bash
claude
```

### 3. Verify it's working

Look for **`woz:code`** on the right side of the text input field:

![woz:code badge](woz-code-badge.png)

That badge means the WOZCODE agent is active.

### 4. Log in

WOZCODE requires a Woz account. On first tool use you'll be prompted to log in, or do it explicitly:

```
/woz-login
```

Or type `/woz` to see all available WOZCODE commands.

This opens your browser to complete sign-in. Credentials are saved and refreshed automatically.

**Headless / SSH?** The terminal prints an auth URL. Open it manually, complete login, copy the token JSON from the success page, and paste it back:

```
/woz-login --token '{"access_token":"...","refresh_token":"..."}'
```

## Usage

Just use Claude Code normally — WOZCODE tools activate automatically. The plugin replaces built-in file tools with smarter versions behind the scenes.

### Agents

| Agent | What it does |
|-------|--------------|
| `woz:code` | Main agent — coding, editing, search, SQL. Auto-delegates to the others when useful. |
| `woz:explore` | Fast read-only codebase exploration (runs on haiku for speed) |

You don't need to switch agents manually. `woz:code` delegates to `woz:explore` as a subagent when it makes sense.

### Commands

| Command | Description |
|---------|-------------|
| `/woz-login` | Log in to your Woz account |
| `/woz-logout` | Clear credentials |
| `/woz-recall` | Recall saved context and preferences |
| `/woz-savings` | Show estimated savings report (roundtrips, time, tokens, cost) |
| `/woz-settings` | Manage WOZCODE plugin settings (attribution, status line, spinner verbs) |
| `/woz-status` | Check authentication status |
| `/woz-update` | Update the WOZCODE plugin to the latest version |
| `/reload-plugins` | Reload plugins to get latest updates |

You can also type `/woz` to see all available WozCode commands in one place.

## /woz-settings

WOZCODE settings live in `~/.claude/settings.json` under the `wozcode` key. Use `/woz-settings` to view or change them — just ask in plain English ("turn off attribution", "hide status line tips", etc.).

| Key | Default | Description |
|-----|---------|-------------|
| `attribution` | `true` | Co-Authored-By on commits + PR badge |
| `statusLine` | `true` | Master toggle for the WOZCODE status line |
| `statusLineSession` | `true` | Show session savings in status line |
| `statusLineLifetime` | `true` | Show lifetime savings in status line |
| `statusLineTips` | `true` | Show quick tips in status line |
| `spinnerVerbs` | `true` | WOZ-themed spinner verbs |

### Commit & PR attribution

When WOZCODE is enabled and you have no existing attribution entry in `~/.claude/settings.json`, WOZCODE installs its own commit/PR co-author line, replacing Claude Code's built-in default. This replacement is announced on the login and session where it first happens. Users who had Claude Code attribution explicitly disabled or customized (any existing `attribution` entry in `~/.claude/settings.json`) are left untouched. Toggle WOZCODE attribution any time with `/woz-settings attribution off`.

Changes take effect immediately. For `attribution`, `statusLine`, and `spinnerVerbs`, also run `/reload-plugins` so Claude Code picks up the change in the current session.

## Managing the plugin

```
/plugin disable woz@wozcode-marketplace     # temporarily disable
/plugin enable woz@wozcode-marketplace      # re-enable
/plugin marketplace remove WithWoz/wozcode-plugin   # remove
```

### Updating

To get the latest version:

```
/woz-update
```

If `/woz-update` fails, force a clean reinstall from your shell:

```bash
claude plugin uninstall woz@wozcode-marketplace
claude plugin install woz@wozcode-marketplace
rm -f "${CLAUDE_CONFIG_DIR:-$HOME/.claude}/wozcode/update-available.json" "$HOME/.wozcode/update-available.json"
```

> **Note:** After reinstalling, quit and restart Claude Code for changes to take effect.

### Debug

To explicitly launch with the WOZCODE agent (not normally needed):

```bash
claude --agent woz:code
```

## Using WOZCODE with Conductor

Connect WOZCODE to [Conductor](https://conductor.build).

1. **Install WOZCODE and restart Claude Code** — follow the install steps above, then restart Claude Code.
2. **Install WOZCODE for Conductor and get the executable path** — from your terminal, run:

   ```bash
   wozcode conductor
   ```

   (or `~/.local/bin/wozcode conductor` if `wozcode` isn't on your PATH)

3. **Paste it** into Conductor → Settings → Advanced → "Claude Code executable path".
4. **Start a new Conductor session** and ask *"what agent are you running?"* — it should answer `woz:code`.

The `claude-woz` executable is auto-refreshed on every WOZCODE session start, so plugin updates keep it pointing at the current version. Not every WOZCODE feature works through Conductor yet, but Search and Edit do — you'll still get most of the speed and cost savings.
