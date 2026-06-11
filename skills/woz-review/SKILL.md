---
name: woz-review
description: Run WozPairProgrammer's deep / before-pushing reviewer on demand. Uses your current Claude Code model over the entire branch diff (pin via wozcode.wozReviewModel or --model). Use when the user wants "a deep review", "final check before pushing", "woz review", or types `/woz-review`.
allowed-tools: Bash(node *), Read
---

# WozPairProgrammer — Deep Review (multi-persona, before-pushing pass)

Run the heavyweight reviewer on the user's current branch. Reviews the full diff vs base (committed + staged + working + untracked) by fanning out to **seven narrow-lens reviewer personas in parallel**, followed by a sequential **wide-lens cross-cutting pass** that sees the narrow personas' findings as priors. Each persona gets pre-fetched knowledge-base context, personal-curation notes, and the slice of CLAUDE.md relevant to its scope.

**Narrow-lens personas (run concurrently):**
1. **Cross-file consistency** — symmetric pairs, rename drift, callers vs callees
2. **Duplication & DRY** — new helpers that should reuse existing utilities, near-duplicate blocks
3. **Codebase reuse & schema-family** — new exports blind to existing repo utilities; sibling Zod schemas that should share a base
4. **Type safety** — `Record<string, unknown>` catch-alls, inline shape casts, null/undefined drift, `||` vs `??`
5. **SDK / library type reuse** — custom types that duplicate Anthropic SDK exports
6. **Correctness & edge cases** — regex semantics, off-by-one, race conditions, error swallowing, security
7. **Comment & docs hygiene** — JSDoc bloat, stale comments after renames, narrative history

**Wide-lens pass (sequential, after the narrow batch):**
- **Cross-cutting & architecture** — trust boundaries (prompt injection), cache-key vs effective-state drift, silent-skip after legacy removal, producer/consumer pairs touched on one side only

**This is the deep pass.** Different from the always-on PostToolUse reviewer:

| | Live pass | Deep pass (this skill) |
|---|---|---|
| Model | Sonnet 4.6 | **your current model** × 7 narrow + 1 wide-lens |
| Turn budget | 10 | 10 per narrow + 20 for cross-cutting |
| Scope | per-edit + branch context | whole branch, 7 specialized lenses + cross-cutting wide pass |
| Auto-patch | yes | read-only by default |
| Trigger | every Edit | manual / every 50 edits |
| Failure mode | full session lost | one persona aborts — the other 6 (and cross-cutting) still finish |

The live pass catches typos, style adherence, and recent-change drift cheaply. The deep pass is what competes with Greptile / human PR review on subtle correctness bugs (regex semantics, multi-file dispatch, error handling), DRY violations cloud bots can't see, and SDK-type drift. Run this before pushing.

## When to use

TRIGGER on: "deep review", "final review", "review my branch", "woz review", "check my work", "is this ready to ship", or `/woz-review`.

## Steps

### 1. Parse user-supplied flags

The user may include flags in their message. Defaults if none specified:
- **read-only** (no auto-patch). Findings are presented, the user decides.
- **markdown report rendered in the conversation** (no file written).

Flags to recognize:
- `--save` — also write the report to `.wozcode/reviews/<branch>-<timestamp>.md`.
- `--apply` — auto-apply high-confidence patch findings (like the live reviewer).
- `--interactive` — emit findings, then ask the user which to apply.
- `--personas <list>` — comma-separated subset of persona ids: `consistency`, `duplication`, `codebase-reuse`, `type-safety`, `sdk-types`, `correctness`, `comments`, `cross-cutting`. Defaults to all eight.
- `--repo <path>` — target a different worktree. Defaults to the session's cwd. Use this whenever the user wants to deep-review a repo other than the one this Claude session is running in (multi-worktree workflows). The branch diff, KB scope, CLAUDE.md, and applied patches all follow the resolved repo root.

If the user is unclear, default to read-only + markdown.

### 2. Invoke the reviewer CLI

Run WITHOUT `2>/dev/null` or any stderr redirect — the CLI streams progress lines to stderr ("`[t+12s] → knowledgeSearch query=…`") so the user sees activity during the wait. Stdout still carries the final markdown report.

```bash
node --no-warnings=ExperimentalWarning ${CLAUDE_PLUGIN_ROOT}/scripts/woz-review.js [--save] [--apply] [--interactive] [--personas <comma-separated-ids>] [--repo <path>]
```

The CLI handles knowledge-base loading, diff packet construction, recursive reviewer invocation, and report formatting. Progress events flow to stderr; the final markdown report goes to stdout.

### 3. Present the report — PRINT IT VERBATIM, FIRST THING

**MANDATORY**: the FIRST content of your assistant response after this CLI returns MUST be the CLI's complete stdout output, copied verbatim. No preamble ("Running the deep review..."), no summary ("Found 3 findings"), no editorial framing ("Here's what the reviewer said"). Just the markdown report itself, full text, exactly as the CLI printed it.

The report already starts with a `# WozPairProgrammer Deep Review` heading and self-describes its sections (summary, findings grouped by file, reasoning trace). It is the user-facing artifact. Do not paraphrase, abridge, or replace it with a summary.

Why this matters: prior dogfood showed the report often gets compressed to a one-line "reviewer surfaced N findings" summary in fresh threads, which silently drops every actionable finding. The user invoked this skill specifically to read the reviewer's verbatim output — anything less defeats the skill.

After the verbatim report, you MAY add (in this order, each optional):
- One short line if `--save` was used noting where the file was written (the CLI's last stdout line already shows this).
- If `--interactive` was used, a follow-up question asking which numbered findings to apply. Otherwise stay silent.

Do not editorialize the findings, evaluate which are most important, or restate them in your own words. The reviewer's wording is the wording.

### 4. Applying findings

If the user replies with "fix", "apply", "do them", "fix all", or names specific finding numbers ("apply 1, 3"), THEN — and only then — execute the patches by calling the woz-edit tool with the patch payloads from the report. The report's `### N. 🔧 Edit` blocks contain the exact `oldString` / `newString` content.

If the user replies with anything else (questions, follow-ups, no reply), don't apply anything. The report is read-only by default.

## Tips

- The reviewer runs against the knowledge base at `~/.woz/knowledge-base/repo/<owner__repo>/kb/`. If the knowledge base hasn't been built for this repo, the reviewer still works but has no PR/code-history context. Mention this once if so.
- This costs Opus tokens per run — typically $2–$8 on a moderately sized branch (6× the single-pass cost, but parallel so wall time is comparable). Each persona has its own 10-turn budget.
- The branch diff includes uncommitted changes — the user can run `/woz-review` mid-edit, not just before push.
- Override the model with `--model claude-sonnet-4-6` if cost matters more than capability for this run.
- Use `--personas` to run a focused subset when you only care about one lens, e.g. `--personas type-safety,sdk-types` after a typing-focused refactor.
- Personal-overlay notes (added via `/woz-knowledge note "..."`) get pre-fetched into the matching personas' prompts. Notes about types feed `type-safety`; notes mentioning "duplicate" or "DRY" feed `duplication`; etc.
