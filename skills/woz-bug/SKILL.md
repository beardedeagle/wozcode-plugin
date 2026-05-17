---
name: woz-bug
description: Report a WOZCODE bug. Same backend as /woz-feedback, tagged for bug triage. Session context (current session id, anonymous id, OS, arch) is auto-attached.
allowed-tools: Bash(node *)
---

# Report a WOZCODE bug

TRIGGER when: user says "report a bug", "woz is broken", "file a bug", or runs `/woz-bug`. For feature requests or general feedback, point them at `/woz-feedback` instead.

If the user already described the bug in their message, use it directly. If they invoked `/woz-bug` with no content (or said something too vague to act on), ask them: "What broke? What did you do, what happened, and what did you expect?" — then wait for their reply before submitting.

Derive `subject` (one-line headline, ~80 chars max) and `body` (the full message, verbatim) from the user's words. Don't paraphrase or add boilerplate.

Submit by piping a JSON envelope to stdin. Use a single-quoted heredoc (`<<'WOZ_FEEDBACK'`) so the shell does NO expansion — user text like `$(cmd)` or backticks is passed through literally and cannot execute. JSON-encode `subject` and `body` so embedded `"`, `\\`, or newlines survive:

```bash
node --no-warnings=ExperimentalWarning ${CLAUDE_PLUGIN_ROOT}/scripts/wozcode-cli.js feedback <<'WOZ_FEEDBACK'
{"type":"BUG","subject":"<json-escaped subject>","body":"<json-escaped body>"}
WOZ_FEEDBACK
```

The CLI auto-attaches `CLAUDE_CODE_SESSION_ID`, anonymous telemetry id (unless the user opted out via `WOZCODE_TELEMETRY_DISABLED=true`), OS release, and architecture. The email is auto-filled from the logged-in account.

On exit 0: tell the user "✅ Bug report sent. Thanks." On non-zero: relay the error verbatim and mention `support@withwoz.com` as a fallback.
