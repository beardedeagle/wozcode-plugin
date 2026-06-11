---
name: woz-knowledge
description: Inspect and customize the WozCode knowledge base — the per-repo + per-user index that powers the reviewer's PR-history and code-file recall. Use when the user wants "knowledge base status", "what's in the knowledge base", "search the knowledge base", "add a note to the knowledge base", "suppress this rule", or types `/woz-knowledge`.
allowed-tools: Bash(node *), Read
---

# WozCode KnowledgeBase — inspect and customize

The WozCode knowledge base has three layers, queried bottom-up:

| Layer | Source | Sync? |
|---|---|---|
| **company** | org-wide chunks (cross-repo learned rules) | server (v2) |
| **repo** | code files + PR history + distilled rules | local (v1) / server (v2) |
| **personal** | your notes, suppressions, boosts | local (v1) / server (v2) |

This skill is a thin wrapper around the `wozcode knowledge` CLI subcommand.
It's the right tool when the user wants to:

- See what's indexed (`status`)
- Search the knowledge base semantically (`query`)
- Add a personal note that the reviewer will surface in future runs (`note`)
- Hide a chunk the reviewer keeps citing (`suppress`)
- Boost the relevance of a chunk that should weigh more (`boost`)
- Ingest a one-off file (`ingest`)
- Refresh the repo knowledge base (`refresh`)
- See your overlay history (`ops`)

## When to use

TRIGGER on:
- "woz knowledge", "knowledge base status", "what's in the knowledge base", "is the knowledge base built"
- "search the knowledge base for ...", "query the knowledge base", "woz recall" (note: distinct from `/woz-recall` which searches past Claude Code sessions)
- "add a knowledge-base note", "remember that X", "tell the knowledge base ..."
- "suppress this", "stop the reviewer from citing X"
- "boost this", "make X more relevant"
- "ingest <file> into the knowledge base", drag-and-drop scenarios
- "refresh the knowledge base", "rebuild the knowledge base"
- `/woz-knowledge` literally

DO NOT use this skill for:
- General-purpose code search — the user's editor / `mcp__plugin_woz_code__Search` is the right tool there.
- Past-session recall — `/woz-recall` searches Claude Code session transcripts, not the knowledge base.
- Running a deep code review — that's `/woz-review`.

## Steps

### 1. Parse the user's request into a knowledge-base subcommand

Map the user's intent to one of the subcommands. When ambiguous, prefer
status + a clarifying question over a guess.

| User says... | Subcommand |
|---|---|
| status / what's indexed / health check | `status` |
| search / find / look for | `query <text>` |
| remember / note / always use | `note "<text>"` (add `--repo` only when user explicitly says "for this repo") |
| remove note / forget note | `unnote <noteId>` (add `--repo` if the note is repo-scoped) |
| suppress / hide / stop showing | `suppress <chunkId>` |
| undo suppress / unhide | `unsuppress <chunkId>` |
| boost / weight higher | `boost <chunkId> <factor>` |
| undo boost | `unboost <chunkId>` |
| ingest / add this file | `ingest <path>` |
| refresh / rebuild | `refresh` |
| my notes / show overlay | `ops` |

For `suppress` / `unsuppress` / `boost` / `unboost`, the user almost
never knows the chunkId off the top of their head. The natural flow is:
1. Run `query <their phrasing>` first to surface candidate hits with ids.
2. Show the user the top hits with their ids.
3. Ask which id to act on, then run the next subcommand.

### 2. Invoke the CLI

Run WITHOUT `2>/dev/null` — stderr surfaces useful errors (login required,
not in a github repo, etc.).

```bash
node --no-warnings=ExperimentalWarning ${CLAUDE_PLUGIN_ROOT}/scripts/woz-knowledge.js <subcommand> [args] [--json]
```

Use `--json` when you need to programmatically inspect output (e.g. to
follow up with another subcommand). Use the human-readable output
when you'll just print it back to the user verbatim.

### 3. Present the output

**For `status` and `query`:** print the CLI output verbatim. Don't
paraphrase the layers / hits — the formatting is already user-facing.

**For `note` / `unnote` / `suppress` / `boost` / `unsuppress` / `unboost`:** print
the one-line confirmation the CLI emits, followed by the new op id so
the user can undo if they want.

**For `ingest`:** print the chunks-added summary.

**For `refresh`:** print the jobId. The local provider's refresh is
fire-and-forget; results land in the next `status` call.

**For `ops`:** if the user is reviewing what they've done, print verbatim.
If they're looking for something specific ("what did I suppress yesterday?"),
filter the JSON output and present a focused list.

## Authentication

Personal-overlay subcommands (`note`, `suppress`, `boost`, etc.) require
login. The CLI surfaces "login required" on stderr; if you see that,
tell the user to run `/woz-login` and retry. Do NOT silently no-op.

## Tips

- the knowledge base is keyed by GitHub origin (`github:owner/repo`). If the
  user is in a non-GitHub repo or a worktree without `origin`, repo-scoped
  ops will skip the repo layer and only operate on personal-global notes.
- The knowledge-base backend (the on-disk store vs the Woz knowledge-base
  server, `'remote'` by default) is an internal setting — it is not configurable
  via `/woz-settings`. Same CLI surface either way.
- `query` runs against every layer the user has access to and merges
  results with overlay ops applied. The `[scope/kind]` tag on each hit
  shows where it came from.
