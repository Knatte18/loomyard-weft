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
    disposition: fixed-differently
    reason: "The proposed HasPrefix(heading, \"## \") on the glossary row alone would have needed the shared comparison to loosen for every row. A levelOnly field makes only the glossary row check the level, and the other rows keep their exact comparison, so no existing assertion is widened."
  - id: F10
    disposition: fixed
---

# Round 1 fixer-report

## Changes

- F1 `internal/summaryparser/description.go`: `mergeDangerSection` skips fenced lines, and the fence lines themselves, both when matching the heading and when looking for the section's end.
  After the self-check, `fenceRun` tracks the opening marker's character and length: a fence closes only on a run of the same character, at least as long, with no text after it.
  `contracts/specs/loom-final-summary-spec.md` states that fenced lines are not read.
  Rows added to `TestValidateDescription_Table`: a section only inside a fence, a fenced example before a real section with a bad door, a shorter fence line inside a longer fence, and a `#` comment line inside a fence in the real section.
- F2 `internal/heddleengine/glossary.md`: dropped the checkout, file name, path and tmux detail from the board, decision record, change description and strand lines.
- F3 `internal/beamcli/cli.go`, `internal/beamcli/doc.go` and `internal/beamcli/describe.go`: the `--draft` help, the package doc and `way_forward` name the Merge danger placeholders.
  `internal/landingshed/doc.go` says the same of the draft, after the affected-set lens found it as the draft's third description.
- F4, F5, F6 `internal/summaryparser/description_test.go`: rows for labels directly under the heading, a level-one heading ending the section, and a door line with a final period.
- F7 `internal/landingshed/beamdescribe_test.go`: a `TestRecordBeamDescription` row records `beamDraft(...)` unedited and wants the two merge-danger findings and no commit.
- F8 `internal/loomengine/discussiontable_test.go`: the chair prompt must contain `strings.TrimSpace(heddleengine.Glossary())`.
- F9 `internal/heddleengine/store_test.go`: the glossary row checks the heading level only, through `levelOnly`.
- F10 `contracts/stencils/loom/loom-template-discussion.md`: the instruction paragraph sits above `{{.glossary}}` and reads "the glossary below".
  The glossary value ends in a newline, so `{{.glossary}}` is followed directly by `## Step 5` to keep one blank line before it; `discussiontable_test.go:243` checks that.

The gate (`lym gate test` over the five touched packages) passes.
Each finding is committed on its own, F4, F5 and F6 together since they share one test file.

## Self-check

- fix-code-correctness, LOW, fence kind and length ignored: fixed with `fenceRun`.
- fix-code-correctness, LOW, the stencil's `{{.glossary}}\n## Step 5` depends on the glossary's final newline: disputed, premise false that it is silent: `internal/loomengine/discussiontable_test.go:243` fails when the blank line goes. No change.
- fix-consistency, LOW, `contracts/specs/loom-final-summary-spec.md` silent on fences: fixed.
- fix-consistency, LOW, fence rule differs from CommonMark's: merged with the first and fixed with it.
- fix-proposal, LOW, nested fence of different length: merged with the first and fixed with it.
- fix-affected-set, LOW, `internal/landingshed/doc.go` omits the draft's placeholders: fixed.

## Disputed

Nothing is disputed against a review finding.
The self-check's glossary-newline finding is answered above with a cited test, and no change was made.

## Deferred

Nothing deferred.

## Rulings

No parent ruling reached this round.
