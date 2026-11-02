# Bench the full suite on OSL-1046 after the binary cuts

`bench-suite-osl1046` · Task · research, gate · running · Worktree-Create

## Goal

The test-suite setup was rewritten to cut the number of test binaries, to get better times on OSL-1046 (this machine, Intel Core Ultra 7 366H, bare-metal Ubuntu, Cortex XDR).
Measure the full suite and add the result to `docs/benchmarks/test-suite-timing.md` as a new measurement report section.

## Cap on runs

- One machine: OSL-1046.
- Three tiers: `tier0`, `tier1`, `tier2`.
- Per tier one cold run, discarded, and three warm runs, the median of the three taken: twelve runs in all, no repeats beyond them.
- No other run, gate use or test suite on the machine during the measurement.

## Method

Follow the method of the latest audit section in the file (`test-time-audit-2`): the gate's summary-line wall for `lym gate test --tier N --all -- -count=1`, with one untracked file in the worktree for the length of the measurement so the gate reads no green record, removed afterwards.
Record machine state (`lym tender status`, `lym gate status`, load average), the per-tier test counts and the packages with the longest `ok` time per tier, as the earlier audits do.
Compare against the earlier audits where the numbers compare, and say plainly where the new gate setup makes them not comparable.
Update the pointer to `gate-test-xdr.md` in the file if the new section changes what it says about OSL-1046.

## Out of scope

No code change, no prune leads filed, no board tasks.
