---
name: woz-feedback
description: Share feedback about WOZCODE — feature requests, general thoughts, anything that's working or not. For broken-behavior reports use /woz-bug (same backend, bug-tagged).
allowed-tools: Bash(node *)
---

# Send WOZCODE feedback

TRIGGER when: user says "send feedback", "share feedback", "i wish woz", "feature request", or runs `/woz-feedback`. For broken-behavior reports prefer `/woz-bug`.

If the user already provided feedback content in their message, use it directly. If they invoked `/woz-feedback` with no content, ask them: "What would you like to share with the WOZCODE team?" — then wait for their reply before submitting.

Derive `subject` (one-line headline, ~80 chars max) and `body` (the full message, verbatim) from the user's words. Don't paraphrase or add boilerplate.

Submit by piping a JSON envelope to stdin. Use a single-quoted heredoc (`<<'WOZ_FEEDBACK'`) so the shell does NO expansion — user text like `$(cmd)` or backticks is passed through literally and cannot execute. JSON-encode `subject` and `body` so embedded `"`, `\\`, or newlines survive:

```bash
node --no-warnings=ExperimentalWarning ${CLAUDE_PLUGIN_ROOT}/scripts/wozcode-cli.js feedback <<'WOZ_FEEDBACK'
{"subject":"<json-escaped subject>","body":"<json-escaped body>"}
WOZ_FEEDBACK
```

The CLI auto-attaches `CLAUDE_CODE_SESSION_ID`, anonymous telemetry id (unless the user opted out via `WOZCODE_TELEMETRY_DISABLED=true`), OS release, architecture, and Node.js runtime version. The email is auto-filled from the logged-in account.

On exit 0: tell the user "✅ Sent. Thanks." On non-zero: relay the error verbatim and mention `support@withwoz.com` as a fallback.
