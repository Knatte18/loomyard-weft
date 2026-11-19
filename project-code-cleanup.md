# P1 Code cleanup: cut and simplify lym code and tests

Rendered 2026-11-19 10:49 UTC; every age below is as of then.
Under: [Board](README.md).

[`code-cleanup`](design-code-cleanup.md) · Age 17h · Owner `neve`

Make lym smaller without changing behaviour: fewer tests, leaner design, no dead compatibility code, no unused features

Make lym smaller without changing what it does, one mechanical pass at a time.

Background: `docs/research/rewrite/shrink-estimate-2026-11-18/report.md` found almost no copy-paste duplication (under 1 %).
The size sits in tests (about 2 test lines per code line), comments,
near-identical helpers a duplicate detector misses, design that makes every caller handle more than it needs,
compatibility and migration code, and features nobody uses.
A clean-room rewrite was judged too large for the gain (2026-11-18); this project cuts in place instead.

## Order

Decided by the operator, 2026-11-19:

1. `drop-compat-migration-code` first, once the single-repo tasks and `hardener-campaign` have landed, since they change the same packages.
2. `doc-go-trim-pass` last, so it trims docs that describe the code after the pass above.

`design-simplify-callers` was dropped (operator, 2026-11-19): its `landingshed-design-simplify` pilot saved 278 lines against an estimate of 600–800.

Tasks are grouped by dependency layer and notes by type.
An entry waits only on the open entries it names under After, so the entries in one layer can run in parallel.

## Tasks (2)

Concrete and claimable; only a task can run.

### Ready (0)

Waits on nothing open; can start now.

_None._

### Draft (2)

Not released; nothing starts it until its status is cleared.

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T1 | `draft` **Remove compatibility and migration code no hub needs any more**<br>• Retired names, key renames and one-off data migrations still in the code; keep the cheap refusals that point to the new name, delete the rest<br>• `loom: code_assist_reviewer_fan review-code`<br>• `loom: code_assist_reviewer_model astra[medium]` | [`drop-compat-migration-code`](design-drop-compat-migration-code.md) | darn · solid | T5<br>T6<br>T10 | 17h<br>31m | fabric, loom, board |
| T11 | `draft` **Trim every package doc.go of what is superfluous**<br>• The comment rewrite did not shrink doc.go files (27,301 lines); cut what is superfluous in all of them, fanned out over many small Sonnet agents<br>• `loom: code_assist_reviewer_fan review-code`<br>• `loom: code_assist_reviewer_model astra[medium]` | [`doc-go-trim-pass`](design-doc-go-trim-pass.md) | darn · solid | T1<br>T10 | 14h<br>0m | docs |

## Notes (0)

Not tasks: ideas and observations, merged into a task when one is promoted.
