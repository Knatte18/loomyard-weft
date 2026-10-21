# Plan review round 4 — fixer report

## Changes

- F1 (BLOCKING), fixed as proposed: card 12's created method is now `Bouncer.startNextGeneration`, in the `**Create:**` line and the case-2 Intent sentence.
  That sentence also states that the existing `startGeneration` keeps writing generation 1's record and is not touched.
- F2 (MEDIUM), fixed as proposed, with one addition:
  - Card 12 `**Edit:**` gains `Bouncer.settledEntry` and `Bouncer.settledGeneration`.
    The Intent says their docs describe a live record over a changed subject, and case 3 over a changed subject, as starting a new generation.
    It also says `settledEntry` reports a new action for them, so `Call` tells them from case 4's clear.
  - Addition: the proposal left the new action unnamed.
    Card 12 now creates `entryNewGeneration` and its Intent says the constant is appended after `entryReplay` to the existing `iota` block.
  - Card 17: `internal/loomengine/anomaly.go` is replaced by `internal/loomengine#ClassifyHalt`.
    The Intent's "`anomaly.go`'s comment on the budget prefix" now reads "`ClassifyHalt`'s doc comment on the budget prefix".
  - Card 18: `loomCLI.reflectHalt` and `loomCLI.loomAfterStep` are replaced by the file glyph `internal/loomcli/halt.go`.
- F3 (MEDIUM), fixed as proposed: card 13 gains a `**Prosa:**` group listing `contracts/specs/shed-recipe-spec.md`.
  Its Intent gains the sentence on the spec's Bouncer-row paragraphs gaining `subject`.
- F4 (MEDIUM), fixed as proposed: card 15's test paragraph adds the direct non-nil assertions.
  These are `RejectReview` in `TestWire_ReviewKeysReachTheEnv`, and `RejectReview` and `Darn.RecordRejection` in `TestWire_DarnRecipe`'s darn-seed subtest.
  The paragraph also says why `NilSeams` cannot report these seams.

`lyx loom validate-plan --overlay` with `{}` reports `ok` after every change, the self-check changes included.

## Self-check

- fix-claims, MEDIUM: the first `entryNewGeneration` signature, `const entryNewGeneration = iota`, would be 0 as a standalone const and would collide with `entryClear`.
  Fixed as proposed: the signature is now `const entryNewGeneration`, and the Intent says it is appended after `entryReplay` to the existing `iota` block.
- fix-consistency, MEDIUM: the same defect as the fix-claims finding.
  Merged with it.
- fix-proposal, MEDIUM: the same defect as the fix-claims finding.
  Merged with it.
- fix-affected-set, outside its lens: the same defect as the fix-claims finding.
  Merged with it.
- fix-consistency, LOW: `internal/shedadapters/doc.go:126` still says a changed subject clears and re-seeds.
  Fixed as proposed: card 12's Intent now names `doc.go`'s Bouncer re-entry cases among the texts it rewrites.
  `doc.go` was already a target.
- fix-affected-set, LOW: the same `doc.go:126` gap.
  Merged with the fix-consistency finding.

## Disputed

None.

## Deferred

- F2's reported-only instance: `contracts/stencils/rubric_test.go`'s file header says a rubric carries the listed markers "and nothing else", which card 13 makes false.
  Fixing it needs that test file under `Edit`, and `edit-test-file` refuses that.
  The review records that the sentence is deferred, so no plan change can carry it.
