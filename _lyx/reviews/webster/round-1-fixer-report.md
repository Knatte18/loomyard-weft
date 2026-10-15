# Round 1 fixer-report

## Changes

- F1: `contracts/stencils/webster/webster-template-master.md` Orientation now says state files are written as ordinary files and read only where the prompt says so.
  The pin in `contracts/stencils/claims_test.go` takes the new phrase and its description; the `{{.plan_dir}}` holds the plan pin is unchanged.
- F2: `internal/websterengine/doc.go` drops the detached line and carries the clause in the opening sentence.
  The rest of that sentence was rebroken at its sentence boundaries, because `lyx loom lint-comments` flagged the mid-sentence breaks the clause would otherwise have left.

`lyx gate test ./contracts/stencils/... ./internal/websterengine/...` and `lyx loom lint-comments` pass.

## Disputed

None.

## Deferred

None.
