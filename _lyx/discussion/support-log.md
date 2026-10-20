# Support log — verify-diff-scope

## Interview

### Turn 1 — scope of the task (auto)

Read the board entry, its decisions of 2026-10-18 and 2026-10-20, and the related entries `dotnet-scoped-verify`, `gate-language-neutral`, `regression-baseline-verify`, `dotnet-verify-protocol`, `darn-verify-tier-unrunnable`, `darn-verify-parent-fix`, `row-entry-gate`, `scratch-defeats-test-cache` and `gate-machine-local-config`.
Question: does tree scope cover loom too?
Recommended and picked: darn only; loom's plan verify is per task and its other gates are scoped.

### Turn 2 — what "at Publish" means (auto)

Code read: darn's Merge-In runs `darn.yaml`'s `verify` on the merged tree, Publish runs only `publish_verify`, Finalize skips on the verified tree.
Picked: the whole suite runs at Merge-In, the landing gate right before Publish.

### Turn 3 — the scoped writer gate (auto)

`internal/impactset` already derives the impacted set plus guard tests for loom's Webster-Burler-Fix round gate.
Question: diff base for darn's gates, last told pass or merge base?
Picked: merge base, so every attempt covers the whole task diff.

### Turn 4 — config shape for tree scope (auto)

Options: config in `darn.yaml`, `language.yaml`, or `landing.yaml`.
Picked: `darn.yaml`, per the board's "config of the module that owns the recipe".
Shape picked: a map tree → target plus a command template with `{targets}`.

### Turn 5 — the scans (auto)

Question: list scan packages in config, or discover them?
Picked: `lyx gate test --guards`, reusing the `//lyx:guard` marker scan, so the list never goes stale.

### Turn 6 — Publish's `publish_verify` and the last gate for a tree-scoped diff (auto)

Picked: a tree-scoped darn run is tree-scoped at every site, and Publish skips `publish_verify` in it.

### Turn 7 — lens review

Seven forks returned; findings merged across lenses:

- bound-breaker BLOCKING: the new keys were settable through the local layer and `run_settings` — fixed, both `CategoryGuard` (D1).
- existing-paths, walkthrough, bound-breaker MEDIUM: `fabricengine.ChangedPaths` drops a rename's old path — fixed, D2 requires a read reporting both sides.
- interactions, walkthrough, bound-breaker: a trailing-slash tree key matches nothing — fixed, trimmed; `.` refused (D1).
- interactions, existing-paths, walkthrough: the Publish skip left the failure record — fixed, the skip takes the unset path and removes it (D4).
- interactions, existing-paths, bound-breaker: `tmux` replay packages unstated for a tree-scoped arrival — fixed, none (D3 step 3).
- bound-breaker MEDIUM: a template unrunnable with empty `{targets}` — fixed, D1 states the requirement in the key's comment.
- bound-breaker MEDIUM: the raw `go test` of the derived command misses the prebuilt `lyx` and the cache — fixed, its test steps run through `lyx gate test` (D3); the remaining cost is an open risk.
- bound-breaker LOW: `--guards` under non-integration tags runs nothing — fixed, refused (D5).
- walkthrough, bound-breaker LOW: the Merge-In skip criterion fails with a failure record — fixed, criterion scoped.
- bound-breaker LOW: tree-scoped Merge-In does not whole-verify merged parent code — fixed, stated in D4's bound.
- goal-scope MEDIUM: `--guards` departs from "scan packages are config" unnamed — fixed, D5 names it.
- goal-scope MEDIUM: the "never on `main`" guarantee ends for tree-scoped diffs — fixed, stated in D2's bound.
- goal-scope LOW: "at Publish" reading, loom exclusion, operator step, measurement — fixed: assumption, Out bullet quoting the Seen sentence, acceptance criterion for the config commands, measurement left to the operator.
- board-fit MEDIUM: `gate-language-neutral` overlap with the `darn.yaml` template — fixed, Out names the expected fold into `language.yaml` tiers.
- board-fit LOW: `--guards` versus `gate-language-neutral`, and the relation to `darn-verify-parent-fix` and `darn-verify-tier-unrunnable` — fixed, named in Out and Open risks.
- leanness: duplicated assumptions, open risks, boilerplate constraints, generic D6 list, Goal restatement, code-shape note — fixed, trimmed.
- existing-paths LOW: `darnReviewBudget` degrades on the new refusals — rejected: run start still refuses, and the status fallback logs a warning as it does for every load error today.

## Rejected alternatives

- Import-graph scope for the last gate: rejected by the operator's 2026-10-18 and 2026-10-20 decisions.
- Tree scope via `impactset` alone: it hard-codes Go and gives a non-Go hub nothing; the board forbids hard-coding Go or a tree layout.
- Tree scope config in `language.yaml`: the board names the module that owns the recipe, and `darn.yaml` already owns darn's `verify`.
- A scan package list in config: goes stale whenever a guard test is added in another package.
- Base the darn gate on the told command's latest pass, as loom's round form does: in darn no whole pass exists before Merge-In, so it would always fall back to the whole command.
- Keep `publish_verify` for tree-scoped diffs: it repeats a module-wide `tmux` tier the scope excluded; a hub that wants it adds it to `tree_scope_verify`.
- Extend tree scope to loom: the plan writer already chooses loom's verify per task; no evidence of need.
- Ship this hub's tree values as the template default: lyx must not hard-code a tree layout.
- Keep `impactset`'s raw `go test` steps: they rebuild `lyx` per `lyxbin` package and miss the warm cache, so the scoped gate could cost more than the whole run.

## Review rounds

- Round 1, APPROVED with eight findings, all fixed; two correct earlier turns:
  `tree_scope_verify` is `CategoryExecutable` and only `tree_scope` is `CategoryGuard`, correcting Turn 7;
  Publish classifies the diff at each call rather than once per run, correcting Turn 6.
  Also `tree_scope` became an open map, a lone `tree_scope` is accepted as scope off, and the `.`-segment and duplicate-key refusals were added (D1).
- Round 2, APPROVED with five findings, all fixed:
  the editor path of `lyx config darn` gains darn's loader check, which it lacked (D1);
  the diff read reports paths verbatim, and an empty-segment tree key is refused (D2, D1);
  the no-`{targets}` refusal covers only a non-empty command, and removing the last tree needs scope off first (D1);
  a non-content guard-scan failure refuses naming its directory (D5).
- Round 3, APPROVED with five findings, all fixed:
  the no-tree refusal covers only a non-empty `tree_scope_verify`, and both keys empty load as scope off (D1);
  the editor refusal keeps the staging copy through its own sentinel and says to copy it out before re-opening the editor (D1);
  the key refusals name the offending keys and leave the way forward to each caller, so a `--set` refusal points at `--set` (D1);
  the classification reads changed paths beside `ChangedPaths`, which keeps its output (D2);
  an unreadable test file arrives as the parse kind with no line, whose way forward covers both causes (D5).

## Question ledger

- Q1 Does tree scope cover loom? — auto-pick: no, darn only (Turn 1).
- Q2 Where does the whole suite run? — auto-pick: Merge-In, right before Publish (Turn 2).
- Q3 Base of darn's scoped gate? — auto-pick: the pair's merge base (Turn 3).
- Q4 Where does tree scope config live, and its shape? — auto-pick: `darn.yaml`, `tree_scope` map plus `tree_scope_verify` template (Turn 4).
- Q5 How are the repo-wide scans named? — auto-pick: `lyx gate test --guards` (Turn 5).
- Q6 Is a tree-scoped diff's last gate whole-module? — auto-pick: no, tree-scoped; Publish skips `publish_verify` (Turn 6).
- Q7 Empty task diff? — auto-pick: tree-scoped with no targets.
- Q8 Template default? — auto-pick: off; the operator sets this hub's values after landing.
- Q9 Derived command's test steps? — auto-pick: through `lyx gate test`, for loom's round gate too (Turn 7).
