---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: scope
    location: internal/loomcli/entryobservation.go:130
    summary: The RecentHistory cap of five rows the entry observation carries has no test, and no test shows a longer history reaching the crash-resume note trimmed to its last rows
  - id: F2
    severity: LOW
    class: consistency
    location: internal/loomengine/config.go:139
    summary: New and changed comment sentences are hard-wrapped mid-sentence at a fixed column, against PATTERN-comment-line-breaks (config.go, entryobservation.go, anomaly.go, friction/doc.go)
  - id: F3
    severity: NIT
    class: consistency
    location: internal/loomcli/entryobservation.go:94
    summary: The three warn messages in observeEntry still say self-report entry observation, a term the diff retired everywhere else
---

### [LOW] RecentHistory cap is not covered by any test

**Location:** `internal/loomcli/entryobservation.go:130` (`recentHistoryRows`, the trim in `observeEntry`)
**Issue:** Card 4 adds `RecentHistory` capped at the last five rows, and `observeEntry` implements the trim.
`TestStep_CrashResumeNoteFollowsTheHandoffVoucher` writes a note from a one-entry history, and `TestNoteCrashResumeAtEntry_WritesNoNote` covers only no-note cases, so a history longer than five never reaches an assertion.
Dropping the trim, or trimming from the wrong end, would pass every test.
This is a behavior the diff adds with no coverage (`PATTERN-test-economy`).
**Fix:** Extend `TestStep_CrashResumeNoteFollowsTheHandoffVoucher`'s existing `MarkerRemoved` case, or its fixture, to a history longer than five rows, and assert the note carries the last five and not the earliest.
No new test function; a row or assertion in the existing test.

### [LOW] New comment sentences are hard-wrapped mid-sentence

**Location:** `internal/loomengine/config.go:139` (`LoomHandoffVoucher`), `internal/loomcli/entryobservation.go:19` (`handoffVoucher`, `recordHandoffVoucher`), `internal/loomengine/anomaly.go` (`EntryObservation.Vouched`, `RecentHistory`, `DetectCrashResume`), `internal/friction/doc.go:11`
**Issue:** `PATTERN-comment-line-breaks` requires one sentence per line for new and changed comments.
The sentences this diff adds or rewrites in these comments break mid-sentence at a fixed column, and several lines carry two sentences; `friction/doc.go` also leaves one overlong line from the reflow ("because self-report is itself a deliberate, closing act the agent never performed. Tier 2 asks every code-touching or").
`start.go`'s new comment in the same diff follows the rule, so the files disagree.
**Fix:** Rewrite only the sentences the diff added or changed, one sentence per line, with a further break at an independent-clause boundary.
Leave untouched legacy lines alone.

### [NIT] Stale "self-report" wording in observeEntry warnings

**Location:** `internal/loomcli/entryobservation.go:94`, `:104`, `:114`
**Issue:** The warns read "for the self-report entry observation".
Card 2 retired the self-report knob and the observation now feeds friction notes.
**Fix:** Say "entry observation".

## Checked and found sound

- Card 1 and card 3 post-edit greps return no match at all, and the card-3 commit records all three files as renames.
- Card 4: `loomAfterStep`, `loomPostRun` and `failedHalt` write one halt note per halt through `reflectHalt`.
  `noteCrashResumeAtEntry` is gated on `c.frictionDir != ""`, and a write failure is warned only.
  The step call sits last in `loomPreStepUnmarked`, so a refused step neither observes nor consumes the voucher, and the `entryObservation` field is gone.
- Card 5: the voucher is written only in the `mustSpawn` branch, after the merge refusal and marker removal and before either arm spawns, gated on Tier 2.
  The three doc comments state the plan's bound.
- Card 6: the scan test mirrors `ghguard_test.go`, uses `NewAllowlist` with `RequireNoStale`, and names the pattern.
  The `PATTERN.md` entry is one line after `PATTERN-github-auth`.
  The stencil keeps its markers and points at the verb for the label.
  `go build ./...` and the tests of the touched packages pass.
