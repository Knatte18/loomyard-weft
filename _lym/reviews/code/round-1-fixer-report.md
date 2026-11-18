---
dispositions:
  - id: F1
    disposition: fixed
  - id: F2
    disposition: fixed
  - id: F3
    disposition: fixed
  - id: F4
    disposition: fixed
  - id: F5
    disposition: fixed
  - id: F6
    disposition: fixed
  - id: F7
    disposition: fixed
  - id: F8
    disposition: fixed
  - id: F9
    disposition: fixed
  - id: F10
    disposition: fixed
  - id: F11
    disposition: fixed
---

## Changes

- F1: `TestResolveRunID` has a row for verb `pr-findings` with a findings-file path, expecting the self run-id and no refusal.
- F2: `writeSendBackNote` writes nothing when `pr-review-send-back-<n>.md` already exists.
  `TestPRFindingsVerb` row `SendBackWhoseNoteAlreadyExistsKeepsTheFirstNote` pre-creates it and expects one file with the earlier content.
- F3: `TestBouncer_Amendment` has `SendBackInputOverAChangedSubjectStartsAFullGeneration` and `AmendmentRecordHoldingAReviewInputRendersNoReviewInputNote`.
  The second row holds a review input in the amendment record, so only the `Amendment` clause of `ReviewInputNote` yields `none`.
- F4: `TestDriftVerify` has two command-mode subtests: a failing compare with an uncaused file, and an exit 0 over an already-ruled file.
- F5: new `internal/driftengine/loosened_test.go` with `TestLoosenedTrailerLines`.
- F6: new `TestPullRequestDeps_ShedStatus` covers the missing run directory and an unparsable status file.
  `TestPRFindingsVerb` has rows for an absent Code-Bouncer target and an erroring entry gate.
- F7: deleted `TestPRReview_WaitSendBackHaltedOnADirtyTreeReroutesOnTheNextCallOnACleanTree`.
- F8: `//testtiming:keep` with its reason above `TestRecipe_PRReviewReroutesToCodeBouncer` and `TestRouting_ProgressAt`.
- F9: `docs/overview.md` words the loosened-check ruling for Drift-Verify and Drift-Recheck, and the ask as once per pull request until a send-back clears it.
  The self-check also found the same stale wording in `internal/landingshed/prreview.go`, `internal/landingshed/doc.go`, `internal/driftengine/doc.go` and `internal/driftengine/verify.go`; each now agrees.
- F10: the fixture comment above PR-Review matches the recipe's wording.
- F11: the fragment header names the includers as a group.

`lym gate test` over `internal/loomcli`, `internal/shedadapters`, `internal/driftengine`, `internal/landingshed` and `internal/loomrecipe` passed at tier 0.

A gate then flagged four comments this round added: two wrapped at a fixed column in `internal/driftengine/doc.go` and `verify.go`, and two lines of 120 characters or more in `internal/landingshed/doc.go` and `internal/loomcli/landingdeps_test.go`.
Each is now one sentence per line.

## Self-check

- fix-code-correctness, LOW: the `AmendmentGenerationRendersNoReviewInputNote` row passed with the `Amendment` clause removed. Fixed (F3's row now holds a review input).
- fix-proposal, LOW: same finding as above, merged. Fixed.
- fix-consistency, LOW: `internal/landingshed/prreview.go` and `doc.go` still said the ask is once per record. Fixed.
- fix-consistency, LOW: `internal/driftengine/doc.go` and `verify.go` omit the loosened-check ruling. Fixed.
- fix-affected-set: no findings.

## Disputed

None.

## Deferred

None.

## Rulings

None.
