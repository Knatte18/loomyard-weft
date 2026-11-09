# Fixer report — round 1, code review

## Changes

- **F1** (fixed as proposed): `internal/boardcli/verbs_test.go` — `stepProjectRelease` with its two-line comment now sits above `stepProjects`, and the `stepProjects` comment block ends directly above its `func`.
  Commit `6a90ee19a`.
- **F2** (fixed as proposed, extended by the self-check): `internal/boardengine/projectrelease_test.go` — `TestSetStatusReleasingProject_OtherWritesReleaseNoMember` gained rows `NonNullStatusOnProject` and `NullStatusOnTask`, and its `write` now returns the released slice, asserted empty for every row.
  The board gained a second draft member `d2`, asserted still draft in every row.
  The unknown-selector case from the finding's summary is not added: the `fix` key does not name it, and it takes the not-found branch before the guard.
  Commit `7632d9ea8`, plus the self-check follow-up below.
- **F3** (fixed as proposed): `TestSetStatusReleasingProject_Refusals` seeds members per row.
  It gained `DoneProjectWithoutDraftMember`, asserting "has no draft member task to release".
  `RunStatusProject` is renamed `RunStatusProjectNamesEveryMember` with two draft members, asserting the exact `"; "`-joined command lines.
  Every member of a row is asserted still draft after the refusal.
  Commit `7632d9ea8`; F2 and F3 share one commit because they edit one file in one round.

## Self-check

- fix-code-correctness, LOW: `NullStatusOnTask` does not pin the `current.Kind != KindProject` half of the guard — fixed as proposed: added row `NullStatusOnDoneTask` (a done task, which would hit the project refusal without the Kind check) and a `d3` done task asserted per row.
  Commit `76ade768d`.
- fix-proposal, LOW: same finding as above — merged, fixed once.
- fix-consistency, LOW: the `TestSetStatusReleasingProject_Refusals` doc comment omitted the no-member message — fixed: the comment now says "or says it has none to release".
- fix-affected-set: no findings.

## Gate repair

The gate refused comment lines of `projectrelease_test.go`: two that reached 120 characters, then a clause-wrapped break in the `TestSetStatusReleasingProject_Refusals` doc comment.
The `TestSetStatusReleasingProject_OtherWritesReleaseNoMember` doc comment is split into sentences.
The `Refusals` doc comment is rewritten as one sentence per line, with no change to its meaning.
Commits `41bbe0cc9` and the one after it.

## Verify failure, no change

The verify run failed tier 2 in `internal/loomcli` on `TestSmokeStatusStrandAcrossDriverSeeds/go_driver`: `TempDir RemoveAll cleanup: ... directory not empty` (`.lym/verify/verify-2.log`).
That is a tmux smoke-test teardown race in a package this change does not touch; the change set is `boardcli` and `boardengine` only.
`lym gate test --tier 2 ./internal/loomcli` passes on rerun (35 passed, 0 failed), so I made no code change.

## Disputed

None.

## Deferred

None.

## Rulings

None.
