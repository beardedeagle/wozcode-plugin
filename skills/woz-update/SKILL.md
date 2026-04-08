---
name: woz-update
description: Update the WOZCODE plugin to the latest version.
allowed-tools: Bash(claude *, rm *)
---

# Update WOZCODE Plugin

Update the WOZCODE plugin to the latest version.

Run these steps in sequence. After each bash command, check the exit code before proceeding.

## Step 1: Update marketplace

Try the update first:

```bash
claude plugin marketplace update wozcode-marketplace
```

If this fails (e.g. git/SSH auth error), fall back to adding via HTTPS, then removing the old entry:

```bash
claude plugin marketplace add https://github.com/WithWoz/wozcode-plugin.git
```

If the add succeeded, remove the old SSH-based entry:

```bash
claude plugin marketplace remove wozcode-marketplace
```

If the add failed, do NOT run remove — the old marketplace entry is still needed. Tell the user: "Marketplace update failed. Check your network connection and try again."

## Step 2: Install latest plugin version

```bash
claude plugin install woz@wozcode-marketplace
```

If this fails, tell the user: "Plugin install failed. Please report this issue at https://github.com/WithWoz/wozcode-plugin/issues"

## Step 3: Clear update flag and confirm

```bash
rm -f "${CLAUDE_CONFIG_DIR:-$HOME/.claude}/wozcode/update-available.json"
```

After all steps succeed, tell the user:
- ✅ WOZCODE updated successfully
- Run `/reload-plugins` to apply the update
