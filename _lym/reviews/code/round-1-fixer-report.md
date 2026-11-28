---
dispositions:
  - id: F1
    disposition: fixed
  - id: F2
    disposition: fixed
  - id: F3
    disposition: fixed-differently
    reason: "The proposed optional group lets the new reader read an old line only. The old pattern requires `skipped \\d+ wall`, so an older reader would still refuse the new line, which the finding also names. The wait moved to stand before `discovered`, where the old patterns' `.*` takes it, so both patterns are back to main's and both directions read."
  - id: F4
    disposition: fixed-differently
    reason: "No held-slot delay is injected. The test sets a stand-in clock on the execution, which reads the slot as held or not and the shared build as run or not, so each wait is an exact literal and no row sleeps or contends for a slot."
  - id: F5
    disposition: fixed
  - id: F6
    disposition: fixed
---

# Round 1 fixer report

## Changes

- F1: the run record gains `slots_held` (`omitzero`, since `omitempty` does not omit a zero `time.Time`).
  `runWithSlots` stamps it once the slots are held or found unneeded, and `SlotWaitSeconds` ends the wait there, falling back to `Start`.
  The description's design decision on the build line is rewritten.
- F2: `report` renders the summary line from a copy of the record holding the call's wait; the rerun's own record still holds none.
- F3 (fixed differently): the line reads `... <scope> wait <s>s discovered ... wall <s>s ...`, and the reader patterns are main's.
  A literal line without the wait field has a row in the `FailedTier` and `TierDiscovered` tables.
  The description's same-binary paragraph is replaced.
- F4 (fixed differently): `TestSummaryLineShowsTheSlotWaitAlone` in `internal/gatecli/slotwait_test.go`, over a slotted run, a carried tier, a shared-build tier, a rerun and a `--copies` run.
  `execution.slotClock` is the seam it uses.
- F5: the fixture spells the new line.
- F6: the reader checks are removed from `TestTierSummaryLine`, with a `Loosened-Check` trailer on the commit.
- The carried-tier expectation in `internal/gatecli/test_integration_test.go` follows the new field position.

Commits: `d6c837e86` (F1, F2, F3, F4, F6), `219ce35a9` (F5), `80c597c16` (self-check, package doc).
The change description under `_lym/landing/summary.md` is edited and not committed.

Checks: `lym gate test` over `internal/gateslot` and `internal/gatecli` passed, and so did the carried-tier scenario at tier 1.
The other tier 1 tests of `internal/gatecli` were not run.

## Self-check

- fix-code-correctness: no findings.
- fix-affected-set: no findings.
- fix-consistency, MEDIUM, the description's report bullet left stale: fixed (merged with fix-proposal's second finding).
- fix-consistency, LOW, the `--copies` prebuild in neither figure: fixed, by stating it in the package doc and the description (merged with fix-proposal's first finding).
- fix-consistency, LOW, the description's title says "beside": fixed.
- fix-proposal, LOW, the `--copies` prebuild in neither figure: fixed, as merged above.
  Counting it in `wall` was not done: `wall` did not hold it before this branch either.
- fix-proposal, LOW, "always agree" contradicts the rerun sentence: fixed, as merged above.

## Disputed

None.

## Deferred

None.

## Rulings

None.
