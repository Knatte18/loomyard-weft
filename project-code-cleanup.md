# P1 Code cleanup: cut and simplify lym code and tests

Rendered 2026-11-18 20:04 UTC; every age below is as of then.
Under: [Board](README.md).

[`code-cleanup`](design-code-cleanup.md) · Age 2h · Owner `neve`

Make lym smaller without changing behaviour: fewer tests, leaner design, no dead compatibility code, no unused features

Make lym smaller without changing what it does, one mechanical pass at a time.

Background: `docs/research/rewrite/shrink-estimate-2026-11-18/report.md` found almost no copy-paste duplication (under 1 %).
The size sits in tests (about 2 test lines per code line), comments,
near-identical helpers a duplicate detector misses, design that makes every caller handle more than it needs,
compatibility and migration code, and features nobody uses.
A clean-room rewrite was judged too large for the gain (2026-11-18); this project cuts in place instead.

Tasks are grouped by dependency layer and notes by type.
An entry waits only on the open entries it names under After, so the entries in one layer can run in parallel.

## Tasks (3)

Concrete and claimable; only a task can run.

### Running (3)

Held by a run; its scope is locked until the run ends.

| # | Task | Slug | Recipe | At | Owner | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- | --- |
| T1 | **List features nobody uses, for the operator to cut**<br>• List features, flags, config keys and verbs with no recent use; the operator decides which go<br>• `loom: code_assist_reviewer_fan review-code`<br>• `loom: code_assist_reviewer_model astra[medium]` | [`drop-unused-features`](design-drop-unused-features.md) | darn · docs | starting · Worktree-Create | neve | 2h<br>0m | tools |
| T2 | **Pilot: simplify the design of landingshed without changing behaviour**<br>• Build the design simplifications the shrink study found in landingshed and measure the lines saved against its estimate, before any such pass on another module<br>• `loom: code_assist_reviewer_fan review-code`<br>• `loom: code_assist_reviewer_model astra[medium]` | [`landingshed-design-simplify`](design-landingshed-design-simplify.md) | darn · solid | Code-Burler-Review | neve | 41m<br>41m | landing |
| T8 | **Comments: check what the big comment rewrite left, before any new pass**<br>• Last week a run of nearly 3,000 agents rewrote docstrings and doc files; measure what is still long before deciding on more<br>• `loom: code_assist_reviewer_fan review-code`<br>• `loom: code_assist_reviewer_model astra[medium]` | [`comment-trim-second-pass`](design-comment-trim-second-pass.md) | darn · docs | starting · Seed-Child | neve | 2h<br>0m | docs |

### Ready (0)

Waits on nothing open; can start now.

_None._

## Notes (2)

Not tasks: ideas and observations, merged into a task when one is promoted.

### Enhancements (2)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| E5 |  | **Design: simplify interfaces that make every caller handle too much**<br>• For example one result type instead of a four-value return handled at 60 call sites; find such spots per module and fix them one module at a time | [`design-simplify-callers`](design-design-simplify-callers.md) | 2h<br>2h | fabric, landing |
| E6 |  | **Remove compatibility and migration code no hub needs any more**<br>• Retired names, key renames and one-off data migrations still in the code; keep the cheap refusals that point to the new name, delete the rest | [`drop-compat-migration-code`](design-drop-compat-migration-code.md) | 2h<br>2h | fabric, loom, board |
