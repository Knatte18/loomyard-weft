# Round 4 fixer report

## Fixed

- **F1** (`91c021a90`): broke every Go comment line listed in the finding at its `, so <clause>` boundary, so the clause that starts with `so` is a line of its own.
  The change touches comments only, in 19 files under `internal/`.
- **F2**: broke the listed markdown lines at their independent-clause boundaries in `docs/overview.md`, `pattern/PATTERN-completion-signal.md`, `contracts/specs/loom-status-spec.md` and `contracts/stencils/orch/orch-template-role.md`.
  `contracts/stencils/claims_test.go` pins one phrase of the stencil on a single line, so its pinned string now carries the new line break.

## Verification

`go build ./...`, `go vet` on the touched packages and the full `go test ./...` pass after both fixes.

## Deferred

Nothing deferred.
The two focus questions on `resume.go` and the `TestResumeVerb` table settled as not-a-defect, so they needed no change.
