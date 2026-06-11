---
description: Knowledge-base tuning for the /woz-review reviewer. `woz-kb tune` is the start-to-finish command — distill human PR comments, backtest the reviewer against historical merged PRs, learn from what it missed, apply to the KB, and re-measure the recall/precision lift — for one repo or every repo in an org. `woz-kb backtest` exposes the individual building blocks (run, --tune, --missed-report, --org-tune, --ab-compare).
---

# /woz-kb — reviewer knowledge-base tuning

One skill, two subcommands:

- **`woz-kb tune`** — the start-to-finish orchestrator: distill → backtest → learn (autotuner new-personas + per-PR missed-fixes) → re-measure the lift. Dry-run by default; `--apply` writes to the KB. Use this to onboard/tune a new repo or a whole org.
- **`woz-kb backtest`** — the building blocks for power users: the raw backtest run plus `--tune`, `--missed-report`, `--org-tune`, `--ab-compare` (unchanged).

The first positional arg selects the subcommand; if omitted it defaults to `backtest` (back-compat).

## When to use

TRIGGER on: "tune the reviewer", "tune this repo", "onboard a repo", "tune the org", "backtest the reviewer", "how well does the reviewer do", or `/woz-kb`.

DO NOT use for: reviewing the current branch (`/woz-review`), past-session recall (`/woz-recall`), or browsing the KB (`/woz-knowledge`).

---

## `woz-kb tune` — start-to-finish

```bash
node --no-warnings=ExperimentalWarning ${CLAUDE_PLUGIN_ROOT}/scripts/woz-kb.js tune \
  --repo with-woz/wozcode \
  --anthropic-api-key-file ~/.woz/.anthropic-backtest-key
```

Pipeline per repo: **(1)** trigger a KB refresh → **(2)** distill human PR comments into baseline persona-hints → **(3)** training backtest (20 PRs × 3 rounds, or reuse a cached run) → **(4)** learn: the autotuner contributes **new personas** (its unique cross-PR synthesis), then missed-fixes contributes the **per-PR durable persona-hints** → **(5)** re-measure on the same PRs and print the recall/precision lift.

Flags:
- `--repo <owner/name>` — tune one repo. **Mutually exclusive with `--org`.**
- `--org <orgId>` — tune every repo the org has indexed, then run a final company-scope org-tune.
- `--apply` — write to the KB. **Without it, `tune` is a dry-run**: it computes + reports proposed hints/personas and counts but writes nothing (and skips the re-measure, since nothing changed).
- `--reuse-run <runId>` — reuse an existing backtest run's PRs + baseline instead of running a fresh (expensive) reviewer pass. `tune` also auto-reuses the newest cached run for the repo when present.
- `--count <n>` / `--rounds <n|all>` — training-backtest size (defaults 20 / 3) when not reusing a run.
- `--skip-distill` / `--skip-refresh` / `--skip-remeasure` — drop individual phases.
- `--anthropic-api-key-file <path>` — use API pricing for the reviewer/judge subprocesses.
- `--judge-model` / `--reviewer-model` / `--reviewer-via` / `--source` — as in `backtest`.

Note: after an `--apply`, the reviewer cache is invalidated (KB changed), so the re-measure re-runs the reviewer on the (small) PR set — a real cost, not a cache hit.

---

## `woz-kb backtest` — building blocks

```bash
node --no-warnings=ExperimentalWarning ${CLAUDE_PLUGIN_ROOT}/scripts/woz-kb.js backtest \
  --repo with-woz/wozcode --count 3 --rounds 1
```

Runs /woz-review against a sample of historical merged PRs and scores how close it gets — the reviewer never sees the human comments or merged diff, so the score is a real recall measure. Key flags:
- `--repo <owner/name>` (required), `--source <path>` (default cwd), `--count <n>` (default 3), `--rounds <n|all>` (per-review-round scoring), `--prs <n,n,n>` (explicit PRs; reuse a prior run's set for a comparable measurement).
- `--tune <runId> [--tune-apply] [--min-apply-ratio <n>]` — autotuner over a finished run.
- `--missed-report <runId> [--tune-apply]` — per-PR missed→suggested-fixes; writes `missed-fixes.json` + `.md`.
- `--org-tune <orgId> [--org-tune-all | --org-tune-repos <list>] [--org-tune-apply]` — cross-repo → company scope.
- `--ab-compare <baselineRunId> <newRunId> [--auto-rollback]`, `--personas <ids>`, `--no-apply`, `--timeout-min <n>`.

## Safety contract (backtest runs)

- A fresh clone per PR at `<repo>/.wozcode/backtests/<runId>/pr-<n>/clone/`; `.wozcode/` is gitignored.
- The clone's `origin` is removed and push URLs are rewritten to `unreachable://` for both `https://` and `git@`; `GH_TOKEN`/`GITHUB_TOKEN`/`GH_ENTERPRISE_TOKEN` are stripped and `HOME` is sandboxed. The reviewer cannot push or authenticate.
- Read-only against upstream. Artifacts persist for inspection.

## Output

Each run writes `<source>/.wozcode/backtests/<runId>/`: `report.md`, `summary.json`, and per-PR/round `reviewer.md`, `findings.json`, `score.json`, `usage.json`. `--missed-report` adds `missed-fixes.{json,md}`; `tune` prints the recall/precision lift.
