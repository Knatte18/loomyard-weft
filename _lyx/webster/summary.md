# claimed-scope-edit: darn record, history gate and PR-Align

All three batches recorded `done`, with no warnings on any envelope.

## Batch 01-discussion-spec (head `6496e6554`)

The record format moved into `loom-discussion-spec.md`, and the Discussion stencil points at it through a specs marker.
`fabricengine` gained `CommittedAnchoredFileHistory`, `discussionparser` gained `CheckAddedDecisionsKept`, and `loomshed` gained the darn discussion check and gate.
The `discussion` gate entry resolves to the darn gate when `Env.DiscussionHistory` is set.
Scope guard deviations: `contracts/stencils/claims_test.go`, `internal/loomengine/discussiontable_test.go`.

## Batch 06-validate-discussion-darn (head `ab6cffd0a`)

A darn run's `validate-discussion` runs the history check and commits the record and support log.
`decision add` works on a darn run once the record is committed.
The darn writer writes the record before any code, and darn's Describe reads it.
Scope guard deviations: `internal/loomcli/decision.go`, `internal/loomcli/decision_test.go`, `internal/loomcli/validate_test.go`.

## Batch 10-darn-review-scope (head `04de13f3c`)

Darn-Review measures against the amended board entry.
The alignment agent's pr-gate measure is a told fragment, and an absent plan reads `none`.
PR-Align is wired with darn's geometry on a darn run.
The darn recipe gained the `discussion` gate, `carry_over: Darn-Review` and a PR-Align row.
The orch role stencil records corrections through `decision add`.
Scope guard deviations: none.

## Verify gate fixes

The first verify attempt failed on `TestGofmt_ModuleFilesAreFormatted` and the fabric-vocabulary tree scan in `TestEnforcement_FabricVocabulary`.
The fixer's reply says it changed three things, all in commit `75724e18b` (`fix: gofmt spacing and bare weft token in the darn record changes`):

- A missing space after a comma in the `composePrompt` signature in `internal/loomengine/prompt.go`.
- The same missing space at three call sites in `internal/loomengine/discussiontable_test.go`; `lyx loom fmt` also reformatted that file further.
- The `decision add` help text in `internal/loomcli/decision.go`, where "reached the weft" now reads "reached the fabric".

The fixer reports `lyx gate test --tags integration` passing for `cmd/lyx`, `internal/lyxcwd`, `internal/loomengine` and `internal/loomcli`, and a clean worktree.
