---
name: woz-share
description: Share a WOZCODE referral code - friends get 20% off their first month, you get $20 in credit.
allowed-tools: Bash(node *)
---

Print the user's WOZCODE referral share message:

```bash
node ${CLAUDE_PLUGIN_ROOT}/scripts/wozcode-cli.js share
```

Relay the full output to the user. Do not summarize or modify it.
