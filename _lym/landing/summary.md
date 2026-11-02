# Measure the full suite on OSL-1046 after the test-binary cuts

Adds a measurement report, `test-time-audit-3`, to `docs/benchmarks/test-suite-timing.md`: the three gate tiers on the work laptop OSL-1046 after the change that builds one test binary per package.
No code changes.

## What changed

- **New section `test-time-audit-3`.**
  It follows the second audit's method: the gate's summary-line wall for `lym gate test --tier N --all -- -count=1`, with one untracked file in the worktree for the length of the measurement and removed afterwards, one discarded cold run and three warm runs per tier, the median of the three taken.
  It records the machine and load, the per-tier walls, top-level test counts and the packages with the longest `ok` time, against the second audit.
- **Pointer line.**
  The OSL-1046 line under "All environments" now names the new section beside `gate-test-xdr.md`; that file is unchanged, since nothing measured contradicts it.

## Results

| Tier | Median wall | Top-level tests |
|---|---|---|
| `tier0` | 27.4 s | 3902 |
| `tier1` | 388.6 s | 937 |
| `tier2` | 124.7 s | 72 |

All twelve runs passed.
`internal/fabricengine` takes 348.7 s of the `tier1` wall; `internal/loomcli`, `internal/reedcli` and `internal/reedengine` bound `tier2`.

## Design decisions

- **Walls are not compared one to one with the second audit.**
  That audit ran on a different machine, without Cortex XDR, and under per-tag binaries.
  The section says so, lists the method differences the binary cuts introduce (the gate's `discovered` and `skipped` totals now include other tiers' tests, and `tier1` runs one `go vet`, not four), and offers the 2026-10-29 OSL-1046 refill row in `gate-test-xdr.md` as the nearest figure, with its scope differences stated.
- **Counts use top-level `PASS` and `FAIL` lines only.**
  Skip lines now mostly belong to other tiers, so counting them would not compare with the second audit.
- **Load was not quiet.**
  No other session ran tests, but the 1-minute load average peaked at 25.84 during `tier1`, against a peak of 11.6 in the second audit.
  The `tier1` figure is reported as measured, with its 25.2 s spread across warm runs.
- **No prune leads and no board tasks**, as the task's scope asks.
