```yaml
lens: code-affected-set
outcome: reported
```

Task: the code-affected-set lens. I traced what each change in the diff forces outside its own lines and listed what the diff leaves wrong.

**Per change: the affected set, covered or not**

- **The red-first gate holds instead of failing** (`NewRedFirstGate` now returns `RedFirstGate`):
  - The one production caller, `internal/shedrecipe/entries_gate.go:353` (`resolveRedFirstGate`), is updated. Covered.
  - The tests in `redfirstgate_test.go` are updated. Covered.
  - Docs updated in the diff: `loomshed/doc.go`, `shedrecipe/doc.go`, `contracts/recipes/darn-recipe.yaml`, `docs/overview.md:1110` and the stencil `darn-template-write.md`. Covered.
  - Docs that only describe the exit codes, not the verdict, need no change: `gatecli/doc.go:855-891`, `gateslot/redfirst.go` and `docs/overview.md:994`. Covered.
  - `contracts/specs/loom-discussion-spec.md:124-132` describes only the record's statement. Not affected.
- **The red-first entry is now MayHold and carries a Final closure.** `shuttleengine` accepts a Pending result and a Final closure on a MayHold entry (`gate.go:93,113`; `wait.go:1834,1878`). Covered.
- **`verifytree.headTree` is now exported as `HeadTree`.** Every reference is renamed: `verify.go:342`, `verify.go:447` and `verify_integration_test.go:679`. No reference to the old name is left. Covered.
- **`preflightTableError`** has one caller, `PreflightStencils` in `darnwiring.go`, and the `wireDarn` doc gains a line. Covered.
- **`clearGateFindings` is removed** from `multillm.go`, `burlerreview.go` and `burlerfix.go`. Updated with it: the `gateattempts.go` header and `gateFailedReason`, the `shedadapters/doc.go` passage, the `BurlerHalfDeps.GateFindingsPath` and `KeepGateFindings` docs, and the tests in `burlerfix_test`, `burlerreview_test`, `multillm_test` and `entries_burlerhalves_test`. Covered. Docs the diff left untouched:
  - `driftengine/deps.go:100` says nothing about removal. Covered.
  - `shedrun/paths.go:301-305`, the `GateFindingsFile` doc, still says the file is removed on a pass. Not covered (F1).
- **`latestDarnOutcome` names the findings file only when the latest entry is a Stuck.** Its doc and `TestLatestDarnOutcome` are updated. Covered. Not covered:
  - `loomshed/doc.go:222` (F2).
  - `loomshed/darnwrite.go:69`, the `DarnOutcome.FindingsPath` doc (F3).
- **Stencil wording** (`darn-template-write.md:110-113`): only the clause about the no-test and unjudged cases changed, from "fails" to "holds". The green-test failure rule, the "fail before the fix" rule and the `No test seam` ask through `lym creel post --parent --expect-reply` are all kept. No rule the answer key left alone is dropped. Covered.
- **Repository rules:**
  - PATTERN-refusal-way-forward: the hold's line and the Final findings both name a way forward, and the Final rows of `TestRedFirstGate_VerdictPerLabelRecordAndExitCode` reach them. Covered.
  - Declared PATTERN changes: the directive declares none. Nothing to land.
- **Commit messages:**
  - `021ac38d`, `4634ff67` and `445885b0` each carry a `Root-Cause:` naming the defect the diff removes: `redFirstFailure`'s exits, `clearGateFindings`, and `wireDarn`'s unconditional wrap.
  - There are no `Pattern-Departure:` trailers.
  - The changed tests now assert the new behaviour that the file is kept; none loosens a check. No finding.

**Findings**

- **F1.** LOW, consistency, `internal/shedrun/paths.go:303-304`. The `GateFindingsFile` doc still says "the row's next call whose gates pass removes it". After the diff no producer removes the file on a pass. Evidence: `internal/shedrun/paths.go:304`, against the removed calls in `shedadapters/multillm.go:191`, `burlerreview.go:561` and `burlerfix.go:334`.
- **F2.** LOW, consistency, `internal/loomshed/doc.go:222`. The package doc says the prior-work note names "the row's kept gate findings file when one exists". After the diff, `latestDarnOutcome` names it only when the latest Darn-Write entry is a Stuck. Evidence: `internal/loomshed/doc.go:222` against `internal/loomcli/darnwiring.go:146-148`.
- **F3.** NIT, consistency, `internal/loomshed/darnwrite.go:69`. The `DarnOutcome.FindingsPath` doc says "empty when none is kept". It is now also empty when the file is kept but the latest entry is not a Stuck. Evidence: `internal/loomshed/darnwrite.go:69` against `internal/loomcli/darnwiring.go:146-148`.

**Tool calls**

1. Grepped for stale "removes" wording, `NewRedFirstGate` and `headTree` callers, and red-first exit descriptions. My context had no repo-wide search for them.
2. Read the red-first lines of `gatecli/doc.go` and `docs/overview.md`, and grepped specs and plugins for red-first. My context lacked the docs outside the diff.
3. Read the red-first lines of `contracts/specs/loom-discussion-spec.md`. The previous grep had named that file.
4. Grepped the `driftengine`, `loomshed`, `shedrecipe` and `loomcli` docs for findings-file keep or remove wording. My context lacked those docs.
