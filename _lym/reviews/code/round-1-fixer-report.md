# Fixer report — code round 1

## Changes

- **F1** (fixed as proposed): `internal/shedadapters/parentseat_test.go` gains a row in `TestBurlerReviewProducer_Call_ParentAskingRule`.
  A focus file one nanosecond past the prior ask's whole-second seed must ask again.
  `TestRoundSeed` pins the `RFC3339Nano` string.
- **F2** (fixed as proposed):
  - Asking rule: rows for an unreadable history and a missing previous report, plus `TestBurlerReviewProducer_Call_ParentRecordOrSeedUnreadable` for a missing round-1 focus file and a corrupt `asks.json`.
    The two read failures live in their own test because the table's tail reads the ask record, which a corrupt file breaks.
  - Coverage gate: rows in `TestParentCoverageGateEntry_Gate` for a done ask whose report is missing (gate error) and a missing review (failing result).
- **F3** (fixed as proposed, in part): dropped the static-prose loop in `TestParentRulesBlock`.
  The blank-line prefix and plain-step equality assertions stay there.
  Folding them into `TestEngine_Review_TableShape` was not done: a fork's grep found no assertion of the block in the two covering tests, so they do not cover them.
  Added `strings.Contains(step, block)` so the test still fails when the block is not composed, and corrected the test's doc comment.
- **F4** (fixed as proposed): deleted `TestParentCoverageGateEntry_Shape`; the parent-review row of `TestEngine_Review_TableShape` asserts attempts above zero and a non-nil gate.
- **F5** (fixed as proposed): `//testtiming:keep` marker on `TestParentCopyPath`.
- **F6** (fixed as proposed): `internal/loomcli/parentseatwiring.go` header says the closures read on a call and the chair's await cap is resolved when the seat is built.

Commits: one per file group, each message naming its findings; F1 and F2 share a file and a commit.
The tier-0 gate for `shedadapters` and `burlerengine` passes.

## Self-check

- fix-code-correctness, LOW, `TestParentRulesBlock` passes with the block never composed: fixed (`strings.Contains(step, block)`).
- fix-consistency, LOW, wiring header line over 119 characters: fixed (split at the semicolon).
- fix-consistency, LOW, stale `TestParentRulesBlock` doc comment: fixed.
- fix-proposal, LOW, nanosecond tests fail on file systems that round timestamps: fixed (`setNanosecondWriteTime` skips there).
- fix-proposal, LOW, wiring header line length: duplicate of the consistency finding, fixed.
- fix-affected-set: no findings.

## Gate repair

The gate flagged the doc comment of `setNanosecondWriteTime` for a line of 120 characters or more.
It is now two sentences on two lines.

## Disputed

None.

## Deferred

None.

## Rulings

None.
