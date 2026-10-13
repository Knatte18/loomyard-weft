---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: consistency
    location: _lyx/plan/10-restore-set-content-rows.md (Intent, the Engine.SetContentRows bullet and the doc.go bullet)
    summary: Card 10 restores the SetContentRows doc comment unchanged and its doc.go paragraph verbatim, both of which claim the pane process retries at its next poll after a refusal, while no pane process reports any more and the card's own added sentence says neither overview pane does
    fix: In the Intent of _lyx/plan/10-restore-set-content-rows.md, change the Engine.SetContentRows bullet from "with its doc comment and body unchanged" to "with its body unchanged and its doc comment unchanged but for its last clause, and the pane process retries at its next poll after any refusal, which is dropped"; and in the doc.go bullet add that the restored paragraph's Bound sentence ends after "so it can resize no strand", dropping "the pane process retries at its next poll after any refusal, which is how a report made before AddStrand has recorded the pane succeeds once it has".
---

### [MEDIUM] Restored comments describe a reporting pane process that no longer exists

**Location:** `_lyx/plan/10-restore-set-content-rows.md`, Intent: the `Engine.SetContentRows` bullet ("with its doc comment and body unchanged") and the `internal/reedengine/doc.go` bullet ("its paragraph comes back").

**Issue:** The code `git show 4ddcb5133` removed carries two sentences about the caller, not the operation.
The method's doc comment ends "and the pane process retries at its next poll after any refusal", and the `doc.go` paragraph's Bound sentence ends "the pane process retries at its next poll after any refusal, which is how a report made before AddStrand has recorded the pane succeeds once it has".
Both described `statuspane`'s reporter, which card 3 (`26604ef69`) deleted under D7; `internal/statuspane` now holds no `Report`, `ReportFromEnv` or retry.
Restoring them unchanged puts a false behavioural claim back into the package doc and the method's doc comment, and the card's own new sentence ("neither overview pane reports a count") then contradicts the paragraph it is added to.
The operation itself, its refusals and its bound are correct and stay; only the retry clauses go.

## Focus settlements

- Card 10 Edit targets: `TestZoomBracket_ZoomedStrandSurvivesEveryReedStep` is listed as a member glyph, and `TestSetContentRows` is a `Create` handle, so no test-file path stands alone for a changed test; `strand_test.go` needs no Edit entry.
  `applyOpts` is a struct gaining a field and `Engine.applyLayoutLockedOpts` keeps its signature (`st *ReedState, live []LivePane, opts applyOpts`), so neither takes the re-sign arrow.
  Not a defect.
- Create placement: both handles use the package unit (`plan:internal/reedengine#…`), which `handle-malformed` requires, and the Intent names `strand.go` (after `Engine.UpdateStrand`, line 643 today) and the end of `strand_test.go`; `strand_test.go` is `package reedengine`, so the handle's unit holds the test.
  Not a defect.
- Fidelity: the rework round is `round-1` (`record.json` carries `class: required` and `first_card: 10`, matching the overview).
  Its one finding F1 is covered by card 10, and each restored piece (method, `InstallPins`, both test parts, `doc.go` list entry and paragraph, budget row) is what F1's fix names; the added `applyLayoutLockedOpts` doc sentence corrects a comment the restored option would falsify, which F1 licenses.
  Scope Out ("content sizing … stay valid for an explicit caller") and D7 ("drops the two overview panes' calls") license the restore; the card keeps the overview panes off it.
  The budget arithmetic holds: the row is 264 today (`4ddcb5133` took it 266 to 265, `17dc83f41` to 264), so one new top-level test makes 265.
- Test economy: no other reedengine test calls `SetContentRows`; `apply_test.go:86` seeds a `ContentRows` value into a plan but never exercises the report path.
  Each of the six rows reaches a distinct branch.
- Reachability: `rows: 0` reaches the pre-lock refusal; `tc:tslug:nobody`, the `below-parent` strand on `%2` and the content strand reported from `%2` reach the unknown-name, not-content-sized and wrong-pane refusals in that order, each named as a `TestSetContentRows` row; no upstream check sits ahead of them, since `SetContentRows` is called in-process.
- Verify coverage: `go test ./...` runs `strand_test.go`; `zoom_integration_test.go` is `tmux && linux`, which the plan vets with `go vet -tags tmux ./...` and the card with `go vet -tags tmux ./internal/reedengine`, which the rubric leaves to Publish.
  Not a defect.
- Attack surface: the Intent states what `InstallPins` lets through (a `SkipFocus` apply installs the window-resized hook array) and what bounds it (focus never moves; only `SetContentRows` sets it, and the name and pane checks bound that operation to the strand's own pane).
  Not a defect.

`lyx loom validate-plan --overlay` with `{}` reports `ok: true`; F1's fix changes no target or verify command, so no further overlay run was needed.
