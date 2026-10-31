# Second recurring test-time and growth audit

`test-time-audit-2` · Task · research, gate

Depends on: [`adopt-check-visible`](design-adopt-check-visible.md)

## Goal

Repeat the recurring test-time and growth audit and append its section to `docs/benchmarks/test-suite-timing.md`, measured the same way as the first audit (`31d9ae96`, 2026-10-28) so the numbers compare.

## Method

- Tiers `tier0`, `tier1` and `tier2`, each through `lym gate test --tier N --all -- -count=1`.
- Per tier: one cold run, discarded, then three warm runs; the median of the three warm runs is the tier's wall.
- That is twelve gate runs in all, and no more: no other tiers, no repeats beyond these.
- Record the load and quiet-machine checks as the first audit did, and name any run that was in flight on the machine during the measurement.

## The section records

- The walls per tier, against the 2026-10-28 audit.
- Top-level test counts per tier and the packages that grew or shrank most since then.
- What `test-prune-audit-leads` (landed 2026-10-29) changed in the counts and walls.
- Prune candidates as lines, as the first audit did; no board task is filed.
