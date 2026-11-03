# Lens review

Every lens fork returned a report; the reports are saved beside this file as `lens-review.lens-<lens>.md`.
Duplicates across lenses are merged and named once with every lens that raised them.

- coverage + way-forward (way-forward 3), MEDIUM, D6 accept path and failing-store resume: fixed differently. Not made: a new driven-run row per approving path; `internal/shedadapters#TestBouncer_Circling_AcceptWithFailedCommitResumesByClearingWithoutApprove` shows the accept path's resume archives the generation without a second approve, so the claimed "resume stores" is false there and a row asserting it would fail. Instead card 7 states that on an accepted round the resume re-enters Discussion-Review and the next approval stores, as for `ApprovePlan`, keeps "resume re-settles and stores" for the converged, skipped and zero-round paths, and lists the accept-path shedadapters tests under `Uses`; the per-path order and failure stay pinned there and the stored bytes by the new real test.
- coverage, MEDIUM, D6 wiring closure untested: fixed as proposed in shape: the closure body is the new `approveDiscussion(location, runID)`, and `TestApproveDiscussion_Real` (replacing `TestStoreDecisionRecord_Real`) drives it, reaching the working-copy read, the `ResolveRunID` slug, the store error and the read error.
- way-forward (4), MEDIUM, done entry holds Discussion-Bouncer forever: fixed as proposed: `approveDiscussion` logs `ErrRecordEntryDone` at Warn and returns nil, per D2's rule that a landed entry keeps its last record; a row of `TestApproveDiscussion_Real` reaches it.
- coverage + row-validity + guard-reach (6), MEDIUM, card 8 darn/beam rows repeat the not-past row and the row argument is never checked: fixed as proposed: the fake's `pastApproval` records the row and answers from `pastRows`, rows set `status.CurrentProducer`, `currentRow` is dropped, and the darn and beam rows are dropped as repeats; the recipe-row rule for darn, sampler and pawl is pinned by card 6's rows.
- coverage + guard-reach (2), MEDIUM, `pastApproval` error in the copy has no stated behaviour: fixed as proposed: card 8 reports it as `record_store_error` naming `decision store`, with a row.
- coverage + guard-reach (5), MEDIUM, card 8 committed-record-missing and read-error branches without rows: fixed as proposed: a row for a committed record HEAD does not hold.
- guard-reach (1), MEDIUM, `--amends-plan` failed-commit refusals lack the store pointer: fixed as proposed: `afterFailedCommit` is re-signed to take the computed `wayCommit` and its three fabric-commit refusals end in it; `rollBackAppend` keeps the bare way since its commit lands no entry; a row in `TestDecisionVerb_AmendsPlan` reaches it.
- row-validity, LOW, card 8 failed-commit rows and amends-plan landed row belong in the existing tables: fixed as proposed: they are rows of `TestDecisionVerb_CommitFailureNamesWayForward` and `TestDecisionVerb_AmendsPlan`, both now under `Edit`.
- row-validity, LOW, nil meaning of the new deps fields: fixed as proposed: card 8 states both are always filled by `preRun` and `decisionFake.deps`.
- coverage + row-validity + guard-reach (3), MEDIUM, card 9 refusals without rows (absent committed record, status/recipe read errors, unresolved parent): fixed as proposed: each has a row, plus the `pastApproval` error.
- row-validity, LOW, card 9 success row needs the committed copy seeded: fixed as proposed: card 9 says rows reaching the copy seed `committedFound` and `committedCopy`.
- way-forward (6), LOW, card 9 empty parent name renders "ask  to run": fixed as proposed: the empty-name refusal names only the unnamed shell, with a row.
- way-forward (7), LOW, card 9 fabric-commit refusal lacks who runs it: fixed as proposed: the clause from `decisionWayFabricCommit` is appended.
- way-forward (9), LOW, card 9 not-past refusal names only `lym loom resume`: fixed as proposed: it also names answering the escalation with the `lym loom circling` verb.
- way-forward (5), LOW, card 8 absent entry gives two ordered ways forward: fixed differently. Not made: suppressing `decisionWayStore` for an absent entry, which would need a second sentinel error; instead `decisionWayStore` reads "once the cause above is cleared, run …", deferring to the board's restore step.
- row-validity + coverage + guard-reach (4), MEDIUM, card 6 non-Bouncer hand-off branch unreached: fixed as proposed: `TestPastDiscussionApproval` gains fixture rows through `discussionApprovalRow`.
- row-validity, MEDIUM, card 5 `TestGetRecord` has no fixture: fixed as proposed: it is a `GetRecord` step of `TestCLI` (tier 1) beside the `--body` steps, storing through `OpenHub` under the fixture's `BOARD_SKIP_GIT`.
- row-validity, LOW, `TestStoreDecisionRecord_Real` spawns a detached sync: fixed as proposed: `TestApproveDiscussion_Real` sets `BOARD_SKIP_GIT`.
- row-validity, LOW, nil-`ApproveDiscussion` row in `TestApproveSeam_FailsToBuild` repeats `NilApproveDiscussionRefused`: fixed as proposed: the row is dropped and the subtest kept.
- row-validity, LOW, card 1 run-held row cannot catch a revision stamp: fixed as proposed: the row advances `*now` before the store.
- row-validity, LOW, `TestDiscussedPayload` as a new test: disputed, every fix worse: the lens itself finds it needs a stored flag that `TestUpsertFieldAllowlist`'s `NewStore("")` stores cannot produce (`internal/boardengine_test#TestUpsertFieldAllowlist`), so a row there cannot reach the stored-value branch.
- coverage, LOW, card 1 unflagged entry with same bytes and `DecisionRecord` read error unrowed: fixed as proposed: two rows added.
- coverage, LOW, card 2 rename of a flagged entry with no file unrowed: fixed as proposed: a row added.
- way-forward (1) + guard-reach (7), MEDIUM, a failed record step skips the render and sync of a saved write: fixed as proposed: the step's error is held while the render and sync spawn run, then returned; the directory-target row asserts the render.
- way-forward (2), MEDIUM, a failed record removal names no way forward: fixed as proposed: the error names the removed entry, the remaining file and to remove it by hand or leave it.
- coverage, LOW, slug operand differs between cards 7 and 8–9: fixed differently. Not made: switching the verbs to `ResolveRunID`, since the decision group's pre-run carries no run id; instead the overview's Shared Decisions names both operands and why they agree for the worktree's own run.
- way-forward (8), LOW, card 3 refusal names no verb: fixed as proposed: it names `lym config board --set design_prefix=design-`.
- forced-edits, LOW, upsert help omits `discussed`: fixed as proposed on card 1, which now edits `internal/boardcli#Command`.
- forced-edits, LOW, `entries_gate.go` comment stale and `shedrecipe/doc.go` target edits nothing: fixed as proposed: card 7 targets `internal/shedrecipe/entries_gate.go` and drops `shedrecipe/doc.go`.
- forced-edits, LOW, card 7 `loomcli/doc.go` under `Edit` while it only adds: fixed as proposed: moved to `Extends`.
- forced-edits, LOW, `fabricengine.BoardDir` comment omits record files: fixed as proposed: card 4 edits `internal/fabricengine#BoardDir`.
- forced-edits, LOW, `doc.go` `# Concurrency and sync` sequence: fixed as proposed: card 2 names it.
