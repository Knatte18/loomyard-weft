---
dispositions:
  - id: F1
    disposition: fixed
  - id: F2
    disposition: fixed
  - id: F3
    disposition: fixed
---

# Code review round 1 — fixer report

## Changes

- **F1** (`41c73284f`): `registerBootstrapTeardown` now registers `waitForFixtureProcessesToExit` before its own kills, so the wait runs after them.
  `TestSmokeWatchdog_StartSpawnsTheDaemon` drops its explicit call.
  This is the review's simpler route.
  The `newWiredPairFixture` bullet of `_lym/landing/summary.md` is rewritten to match.
- **F2** (`ee02e99f5`): the cleanup keeps the set of killed pids, kills any new one on each scan past the deadline, and fails only when every pid left was already killed.
  The helper's doc comment and the summary's first bullet say so.
- **F3** (`f8b22913c`): added the `//testtiming:keep` line to the new test.
- **Self-check** (`f8b22913c` is followed by an empty commit): the `Loosened-Check:` trailer for the F2 change.

Gate: `lym gate test --tier 1 --tier 2 ./internal/loomcli` passed both tiers with no failures.
It ran on the tree that holds all three fixes, before they were committed one by one.

## Gate finding after the report

The verify run (`.lym/verify/verify-3.log`) exited 1 in tier1:
the memory guard killed the `cmd/lym` test binary at 1199 MB, when the user slice had 1516 MB left of 16384 MB.
The log reports no failed test.

Repair: no code change.
The branch changes only test files in `internal/loomcli`, none of which is part of the `cmd/lym` test binary.
`lym gate test --tier 1 ./cmd/lym` on the same commits then passed with no failures.
My reading is that memory pressure on the machine's user slice caused the kill; I did not measure what else held that memory.
If the guard kills this binary again, the settings to look at are the machine's load or `gate.memory_quit_mb`, which only the prime changes.

## Self-check

- fix-code-correctness: no findings.
- fix-consistency: LOW, the F2 commit loosens the cleanup's failure condition with no `Loosened-Check:` trailer — fixed as proposed, in a later empty commit.
  The trailer's cause comes from reading the loop; no run measured it.
- fix-proposal: no findings.
- fix-affected-set: no findings.

## Disputed

## Deferred

## Rulings
