---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: design
    location: internal/loomcli/decision_test.go:140
    summary: the lazily resolved LOOM program name has no test where the resolver fails, so noticeFits passing on a geometry error and decision add not aborting are unpinned
    fix: add a row to the decision-add table in internal/loomcli/decision_test.go whose decisionDeps.programName returns an error; assert that a one-entry add is recorded and that noticeFits does not refuse.
    origin: lens:code-tests
---

### [LOW] Failing program-name resolver is untested

**Location:** `internal/loomcli/decision_test.go:140`, `internal/loomcli/decision.go:304-311`.
**Issue:** The fix that resolves the LOOM program name lazily changed two behaviours: `decision add` no longer aborts on a geometry or `FormatProgram` failure, and `noticeFits` returns true when `programName()` errors.
The only test reference to `programName` always returns `("hub:LOOM", nil)`.
Reverting to the eager lookup, or flipping the error branch to `false`, would break no test.
The code itself is correct: the notice posted as the program fails on the same geometry and is reported per strand.

## Rejected

None.

The alignment report's claims hold against the diff.
The reed pane launch line still wraps the shuttle launch command, which carries `--append-system-prompt-file`, and `internal/reedengine/doc.go` holds both the parent's spawn-refusal line and the `CreelHooks` paragraph.
`PATTERN.md` and `pattern/` are unchanged by the merge.
The `internal/loomcli` darn wiring change does not touch the decision-notice cap.
The probe change in `internal/creelengine/deliver.go` and its revert net to zero.
