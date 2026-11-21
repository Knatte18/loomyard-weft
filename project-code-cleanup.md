# P1 Code cleanup: cut and simplify lym code and tests

Rendered 2026-11-21 19:30 UTC; every age below is as of then.
Under: [Board](README.md).

[`code-cleanup`](design-code-cleanup.md) · Age 3d · Owner `neve`

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

## Tasks (0)

Concrete and claimable; only a task can run.

### Ready (0)

Waits on nothing open; can start now.

_None._

## Notes (0)

Not tasks: ideas and observations, merged into a task when one is promoted.
