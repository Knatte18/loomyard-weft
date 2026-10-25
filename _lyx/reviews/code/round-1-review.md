---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: scope
    origin: lens:code-pattern-fit
    location: internal/loomcli/cli.go:489
    summary: the lyx loom help example still shows the removed `lyx loom review approve <slug>` verb
    fix: in the `Long` help text of the `lyx loom` command at internal/loomcli/cli.go:489, replace the example line `lyx loom review approve <slug>` with a live verb such as `lyx loom circling accept <slug>`, and update any test that asserts the help text.
  - id: F2
    severity: LOW
    class: scope
    origin: lens:code-pattern-fit
    location: internal/shedrun/paths.go:242
    summary: shedrun.AlignmentDir and AlignmentRel and the status-commit staging of the alignment directory outlive the deleted alignment agent
    fix: delete AlignmentDir and AlignmentRel in internal/shedrun/paths.go, the alignment staging at internal/loomcli/wiring.go:111 with its comment at wiring.go:79, the related case in wiring_commitstatus_test.go and the AlignmentDir line in internal/shedrun/doc.go:30. If staging directories an older binary left is wanted, keep the staging and reword its comment to say so.
  - id: F3
    severity: LOW
    class: scope
    origin: lens:code-tests
    location: internal/reportengine/archivedrecords.go:1
    summary: the read-only decoders of archived parent-review and alignment records have no direct test, and no fixture covers attempt.json or delivery.json
    fix: add an archivedrecords_test.go beside the decoders (or rows in the existing reportengine fixtures) that decode one record of each on-disk shape (request.json, delivery.json, verdict.json, alignment.json, attempt.json, verdict-<token>.yaml) and assert the counts the reports read from them.
---

Range reviewed: `git diff dd11ab09f..HEAD`, nineteen commits, one per plan card.
The verify gate decides build and test results; none are re-derived here.
No landing summary was present.

### [MEDIUM] Removed verb still advertised in the lyx loom help

**Location:** internal/loomcli/cli.go:489
**Issue:** D2 removes the `lyx loom review` verb tree, but the root command's help example still lists `lyx loom review approve <slug>`.
An operator copying it gets an unknown-command error.

### [LOW] Alignment directory helpers and staging outlive the alignment agent

**Location:** internal/shedrun/paths.go:242
**Issue:** D1 deletes the alignment agent whole.
`AlignmentDir` and `AlignmentRel` have no remaining writer; their only users are the status commit's pathspec and one test.
The sibling parent-review directory was dropped from the same staging, so the difference has no stated reason.

### [LOW] Archived-record decoders in reportengine lack direct coverage

**Location:** internal/reportengine/archivedrecords.go:1
**Issue:** D4 keeps reports counting archived records through new decoders.
Existing report fixtures reach some shapes only indirectly, and none builds `attempt.json` or `delivery.json`, so drift in those decoders passes silently.

## Holistic and focus notes

Each focus item was settled, with no further defect:

- **Part 1 removals:** no non-test reference to `internal/alignment`, `internal/parentreview`, `lyx loom review` or the four retired keys remains outside the D4 decoders.
  Reconcile drops each retired key with a `<key>: retired, removed` line (`internal/loomengine/config.go:1056`), and a run-settings test covers the read-time migration.
- **D8:** `routeDecisions` runs ahead of `judgmentRefusal` in `settleUnlessRefused`.
  An open decision escalation makes the round neither a settled generation nor a recorded stand, so a CONVERGED round re-returns the same Awaiting.
  `retireContinue` archives the escalation record, so the held-refusal re-escalation re-renders its brief.
  The hard-cap halt carries a budget-exempt `Halt` and names `lyx loom resume`.
  The accept carries every decision finding with the `Added after Discussion` titles recorded since the snapshot.
- **Row keys and arbiter:** `escalate_decisions` and `decisions_escalated` are set together on the two Discussion rows only, pinned by `decisionswitches_test.go`; `arbiterstop` refuses a `decision` escalation and a decision halt.
- **Lenses:** `goal-scope` and `board-fit` each end in their own list and read at most six calls; `review-discussion` gains both after its existing lenses; `twill-discussion` is unchanged.
- **Stencils:** the rubric defines `decision` and corrects the "Class never changes" sentence; the discussion and darn writer stencils carry D9 and D10; no rule the record did not ask to change was dropped.
- **Part 3:** fabric's own merge-in names are untouched.
  `LoomRetiredRows` carries a `Merge-In` status to `Parent-Align` for every loom verb, and `loomcli/wiring_darn_test.go:276` covers it.
  The fixer-attempt and fixer-report fallbacks are tested (`landingshed/parentalign_fixer_test.go:503`, `loomcli/subjects_test.go:143`).
  The remaining `Merge-In` strings are retired-name handling, or stale test names and comments that stay inside the verify gate's scope.

## Rejected

- Lens code-affected-set F1 and lens code-pattern-fit F3, `internal/creelengine/doc.go:202` names the parent-review gate: the sentence describes the future `creel-agents` stage, which the record's Scope Out leaves to that note.
- Lens code-tests F1, no test for the old fixer-attempt record fallback: `landingshed/parentalign_fixer_test.go:503` reads the record under `retiredMergeInName`.
- Lens code-tests F2, no test for the old fixer-report fallback: `loomcli/subjects_test.go:143` and `:175` exercise the `Merge-In-fixer-report.md` path.
- Lens code-tests F5, no test that a `Merge-In` status resumes on `Parent-Align`: `loomcli/wiring_darn_test.go:276` runs `LoomRetiredRows()` through the rename.
- Lens code-tests F4, no read-time test for a profile holding `merge_fix_attempts` or the `merge-in` gate key: the read-time path runs the same `MigrateConfig` that `TestMigrateConfig` in `landingshed` and `gateslot` covers; no distinct uncovered behaviour was shown.
