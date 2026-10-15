## Fixer-report, round 1

### Changes

- F1: `contracts/stencils/loom/loom-rubric-plan-review.md` now starts "- **The re-sign arrow.**" on its own line after the Member glyphs bullet. Fixed as proposed.
- F2: `contracts/stencils/loom/loom-template-plan.md` no longer has the older "A new test function in an existing file..." and second "A new test file is a `**Create:**` path." sentences.
  The `wantAll` claim in `contracts/stencils/claims_test.go` now pins "A new test function is a `**Create:**` declaration with a `plan:` handle"; the "A new test file is a `**Create:**` path" claim still matches the kept sentence. Fixed as proposed.

The F2 claim edit had dropped the space after a comma in `contracts/stencils/claims_test.go:258`, which failed `TestGofmt_ModuleFilesAreFormatted` at the verify gate.
I restored it with `lyx loom fmt` and committed it separately.

`lyx gate test ./contracts/stencils` and `lyx gate test ./cmd/lyx` pass.

### Disputed

None.

### Deferred

None.
