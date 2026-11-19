# P1 Code cleanup: cut and simplify lym code and tests

Rendered 2026-11-19 09:31 UTC; every age below is as of then.
Under: [Board](README.md).

[`code-cleanup`](design-code-cleanup.md) · Age 16h · Owner `neve`

Make lym smaller without changing behaviour: fewer tests, leaner design, no dead compatibility code, no unused features

Make lym smaller without changing what it does, one mechanical pass at a time.

Background: `docs/research/rewrite/shrink-estimate-2026-11-18/report.md` found almost no copy-paste duplication (under 1 %).
The size sits in tests (about 2 test lines per code line), comments,
near-identical helpers a duplicate detector misses, design that makes every caller handle more than it needs,
compatibility and migration code, and features nobody uses.
A clean-room rewrite was judged too large for the gain (2026-11-18); this project cuts in place instead.

## Order

Decided by the operator, 2026-11-19:

1. `drop-compat-migration-code` first, once `landingshed-design-simplify` and `single-repo-fabric-mode` have landed, since both touch the same packages.
2. `design-simplify-callers` next, module by module, after the `landingshed-design-simplify` pilot shows whether the estimate holds.
3. `doc-go-trim-pass` last, so it trims docs that describe the code after the two passes above.

Tasks are grouped by dependency layer and notes by type.
An entry waits only on the open entries it names under After, so the entries in one layer can run in parallel.

## Tasks (0)

Concrete and claimable; only a task can run.

### Ready (0)

Waits on nothing open; can start now.

_None._

## Notes (3)

Not tasks: ideas and observations, merged into a task when one is promoted.

### Enhancements (3)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| E1 |  | **Trim every package doc.go of what is superfluous**<br>• The comment rewrite did not shrink doc.go files (27,301 lines); cut what is superfluous in all of them, fanned out over many small Sonnet agents | [`doc-go-trim-pass`](design-doc-go-trim-pass.md) | 13h<br>13h | docs |
| E5 |  | **Design: simplify interfaces that make every caller handle too much**<br>• For example one result type instead of a four-value return handled at 60 call sites; find such spots per module and fix them one module at a time | [`design-simplify-callers`](design-design-simplify-callers.md) | 16h<br>1h | fabric, landing |
| E6 |  | **Remove compatibility and migration code no hub needs any more**<br>• Retired names, key renames and one-off data migrations still in the code; keep the cheap refusals that point to the new name, delete the rest | [`drop-compat-migration-code`](design-drop-compat-migration-code.md) | 16h<br>2h | fabric, loom, board |
