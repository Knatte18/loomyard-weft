# Decision record — record-batch accepts a Create target declared in the unit's external test package

## Goal

`lyx webster record-batch` stops failing a batch with `create-not-done` when a Go Create target `<dir>#<Member>` was written into the external test package `<dir>_test`, or the reverse, so a plan naming a test on either package of a directory can record clean.

## Scope

- `internal/planglyph`'s `DoneChecks`: a Go Create target or Rename new side answered not_found counts as done when the same member is declared in the other package of its directory.
- `internal/planglyph`'s `BindHandles`: a handle whose expected glyph matches no delta symbol binds to the same member in the other package of its directory, so the plan's handle is rewritten to the glyph the card created.
- `internal/planparser`: one exported helper mapping a Go member glyph to the same member in the other package of its directory.
- The glyph-conversion chokepoint's background file names the helper as a sanctioned external-test unit mapping.
- Out of scope: a plan-time validator rewriting such targets, and C# plans.

## Decisions

### Resolve across both packages of a Go directory at record time

Decision: the done-check and the handle binding accept the member in the counterpart package, `<dir>` for `<dir>_test` and `<dir>_test` for `<dir>`, when the target itself is missing.
Rationale: at plan time the external test file may not exist, so no validator can know which package a new test lands in; at record time the declaration is on disk.
Bound: a target passes only when a declaration of the same member name exists in the same directory's other package, under a plan whose language is go; a member declared anywhere else still blocks, and C# plans are unchanged.

### The done-check reads the counterpart from the declarations listing it already loads

Decision: `DoneChecks` matches the counterpart glyph against the worktree declarations it lists for a missing member, rather than issuing a second resolve.
Rationale: the listing is already fetched exactly when a Create target is not_found, and it names the external test package's declarations.

### The binding rewrites the handle to the counterpart glyph

Decision: `BindHandles` binds a handle matched through its counterpart to the counterpart glyph.
Rationale: the plan's later refs then name the glyph the card created, so drift detection and later batches resolve it.

### One helper owns the counterpart mapping

Decision: `planparser` exports the counterpart mapping beside `goPackageDir`, built on `ParseRef` and `Glyph.String`.
Rationale: the glyph-conversion chokepoint confines Go unit adjustments for the external-test pseudo-unit to `planparser`.

## Constraints

- `PATTERN-glyph-conversion-chokepoint`: the unit adjustment lives in `planparser` and goes through `glyph.Parse` and `Glyph.String`.
- `PATTERN-test-economy`: new cases are rows in the existing done-check and binding tests.
- `PATTERN-comment-line-breaks`: one sentence per comment line.

## Auto-mode assumptions

- The counterpart check applies in both directions, since a plan may equally name `<dir>_test` for a test written in-package.
- Rename new sides get the same treatment as Create targets, since they share the not_found path.

## Open risks

- A real package directory whose own name ends in `_test` maps to its sibling directory's package, as `goPackageDir` already does; the counterpart then only passes when that sibling declares the member.

<!-- lyx:carry-over Code-Review -->
- Code-Review, round 1: converged; each finding below was fixed or disputed by the round's fixer, and no fresh reviewer has seen the result; the fixer report says which.
  Review: _lyx/reviews/code/round-1-review.md; fixer report: _lyx/reviews/code/round-1-fixer-report.md.
  - rename-new-side-counterpart-untested (class scope, severity MEDIUM, open in rounds 1)
<!-- /lyx:carry-over Code-Review -->

## Acceptance criteria

- A Go Create target `<dir>#TestX` whose member is declared only in `<dir>_test` raises no `create-not-done`.
- A Go Create target `<dir>_test#TestX` whose member is declared only in `<dir>` raises no `create-not-done`.
- A Create target whose member is declared in an unrelated unit still raises `create-not-done`.
- A Create handle `plan:<dir>#TestX` whose delta created `<dir>_test#TestX` binds to `<dir>_test#TestX` with no `bind-count-mismatch`.
- A C# plan's Create target gets no counterpart lookup.
