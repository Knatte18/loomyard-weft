# Fixer-report, code review round 1

## Changes

- **F1** (fixed as proposed, one deviation in shape): `internal/battencli/startguard.go` replaces the `Draft func() (bool, error)` seam with `Unreleased func() (string, error)`, which reads `draft` or `discussed` for a task and `""` otherwise.
  The refusal reads "`<slug>` is a `<status>` task, which no run starts" and keeps the release command as the way forward.
  The field, `refuse` and release-helper comments name discussed beside draft.
  `startguard_test.go` gains a discussed-task refusal row, and the draft rows use the new seam.
  The seam took the status as a string instead of a second bool so the refusal can name the status the entry carries, as the finding asked.
- **F2**: `TestSetRunStatus` gains a row seeding `running · Webster` and calling `SetRunStatus` with the same status.
  The assertion differs from the proposal: a skipped write is observed through the rendered `Home.md`, removed before the call, staying absent, because a byte comparison of `board.json` passes with the skip clause deleted (a fork showed this, and I confirmed the new row fails with the clause removed).
  The proposed byte comparison and the `Revised` seed are the changes not made.
- **F3**: `commitstatus_test.go` gains `TestNewCommitStatusSeam_BoardWriteFailureDoesNotStopTheCommit` and `TestBattenCommitStatusDeps_SetBoardStatusWritesTheRunsEntry`.
  The second drives the real closure over a Tier 0 hub from `boardkit.HubWithBoardConfig`, not a hubforge fixture, as `TestWire_CreateWorktreeClaimsTheBoardEntry` does.
- **F4**: `run_commitstatus_test.go` gains `TestStep_StartWriteWithAnEntryGatedFirstRow`, with a holding-gate row and a skipping-gate row.
- **F5**: `verbs_test.go` gains `stepListShowsADiscussedEntry`, registered in `cli_test.go`, asserting layer `Discussed` in the JSON list and a `[discussed]` suffix on the `--text` line.
- **F6**: `layer_test.go` gains a draft-with-discussed-flag row expecting `Draft`, and `TestRenderDiscussedSection` asserts the Recipe and After columns inside the Discussed section.
- **F7**: `boardengine/doc.go` lists `discussed` among the non-run statuses, and `boardcli/cli.go` help says null releases a draft or discussed task in the set-status text, its status field and the merge field.

Gate: `lym gate test` over `battencli`, `boardengine`, `boardcli` and `shedengine` at tier 0, and over `boardcli` and `battencli` at tier 1, passed before the self-check, and `boardengine` at tier 0 passed again after the F2 change.
Each finding is one commit; the F2 follow-up is a second commit.

## Self-check

- fix-code-correctness, MEDIUM: F2 row passes with the skip clause deleted. Fixed with my own change (observe `Home.md`), verified by deleting the clause and seeing the row fail.
- fix-proposal, MEDIUM: same finding as above, merged. Its alternatives (modification time, Bolt commit count) were not used; the rendered-file check needs neither a clock nor git.
- fix-consistency: no findings. It noted `internal/battencli/cli.go:53` saying a beam project "skips the draft check", still accurate for the beam case, so left as is.
- fix-affected-set: no findings.

## Disputed

None.

## Deferred

None.
