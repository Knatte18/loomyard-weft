# Code cleanup: cut and simplify lym code and tests

`code-cleanup` · Project · enhancement

Make lym smaller without changing what it does, one mechanical pass at a time.

Background: `docs/research/rewrite/shrink-estimate-2026-11-18/report.md` found almost no copy-paste duplication (under 1 %).
The size sits in tests (about 2 test lines per code line), comments,
near-identical helpers a duplicate detector misses, design that makes every caller handle more than it needs,
compatibility and migration code, and features nobody uses.
A clean-room rewrite was judged too large for the gain (2026-11-18); this project cuts in place instead.

## Order

Decided by the operator, 2026-11-19:

1. `drop-compat-migration-code` first, once `landingshed-design-simplify` and `single-repo-fabric-mode` have landed, since both touch the same packages.
2. `design-simplification` next, module by module, after the `landingshed-design-simplify` pilot shows whether the estimate holds.
3. `doc-go-trim-pass` last, so it trims docs that describe the code after the two passes above.
