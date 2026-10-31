# Record the second recurring test-time and growth audit

## What changes

`docs/benchmarks/test-suite-timing.md` gains a `test-time-audit-2: second recurring audit` section after the first audit's Correction block.
It re-times `tier0` to `tier2` by the first audit's route and method, so the walls and counts compare: `lym gate test --tier N --all -- -count=1`, one discarded cold run and three warm runs per tier, the median of the warm three as the wall.
No code, test or PATTERN entry changes.

## What the section records

- Walls against the 2026-10-28 audit: `tier0` 18.8 to 19.6 s, `tier1` 58.8 to 62.9 s, `tier2` 50.0 to 50.5 s.
- Top-level counts: `tier0` 3734 to 3769, `tier1` 902 to 907, `tier2` 69 to 70.
- Packages that grew or shrank, counted by `func Test` declarations at `ba2a346b` and at the audited revision.
- The effect of `test-prune-audit-leads` (`4a059d2a`): one test fewer, and `internal/reedengine` in `tier2` from 40.00 s to 12.75 s with the `tier2` wall unchanged, since `internal/loomcli` and `internal/reedcli` bind it.
- Prune candidates as lines, no board task filed.

## Design decisions

- Per-package counts come from `git grep` over both revisions, not from the gate logs, because the first audit's Correction block retracted the log-based split; the gate run count stays at twelve.
  The declaration counts agree with the gate's tier totals only roughly (+43 against +41), and the section says so.
- The machine was busier than in the first audit: load 3.04 before the first run, and `landing-watch` ran scoped gate runs during the last `tier1` run and two `tier2` runs.
  The section names those runs and states the `tier1` and `tier2` walls as upper bounds; no run was repeated, as the entry allows no more than twelve.
- The effect of `test-prune-audit-leads` is read from the commit's diff and the two audits' package times, since a before-tree measurement would exceed the twelve runs.

## Testing

`lym gate test --wait` passed on the diff.
The figures come from the twelve gate runs; the gate's logs and records are under the hub's gate runs directory.
