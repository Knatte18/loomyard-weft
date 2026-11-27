# Lens review

Every lens fork returned a report; each is saved beside this file as `lens-review.lens-<lens>.md`.
Duplicates across lenses are merged into one line, naming each lens that raised it.

## Findings

- coverage 1, guard-reach 1, way-forward 3, forced-edits (outside lens) — BLOCKING — fixed as proposed: a drop moves the landing's records commit into the run's pair entry as `HeldRecords` (cards 4, 11), `HeldRecordsCommit` and `PushRun` read it (cards 4, 9), and `RemovePair` removes it with the pair (card 19); the overview's shared decision says the hold outlives a dropped landing.
- coverage 2, guard-reach 2 — BLOCKING — fixed as proposed: card 9's `PushRun` sets `RecordsHold` only when `l` is the run's own pair, pushes the prime's records whole, and gains a prime-`l` row.
- guard-reach 3 — BLOCKING — fixed as proposed: card 9 adds `PairPush.CodeCeiling`, which `PushRun` sets for a beam pair holding landings at the oldest landing's first parent, and card 11's beam `PushTip` passes the judged tip; `lym fabric push` stays the operator's whole push, bounded on card 9 as the way forward D5 names.
- way-forward 1 — BLOCKING — fixed differently: the proposed clear-on-rebase re-judge was not made alone; card 11's rebase refusal now names the rebase then the parent target's push, and `SettleRefused` re-reads each landing's tip from `LandingCommits` (card 10) so the hand-pushed rebased tip settles and clears the record.
- way-forward 2 — BLOCKING — fixed as proposed: `SettleRefused` runs the drop read (step 2) as well as the settle, so an owner's landing under another commit drops the landing and clears the record.
- coverage 3, guard-reach 4, way-forward 7 — MEDIUM — fixed differently: the settle-only pusher is gone; `TargetRefused` (card 6, landing read on card 11) gates the starter and the conditions so a refused target starts no pusher, and the dobby tick runs `SettleRefused` in-process through `restartHubPushers`; `TestStart` and `TestRestartHubPushers` pin both.
- way-forward 4 — MEDIUM — fixed as proposed: the unrecorded-landing refusal names `lym loom resume` in that run's task worktree or a rebase dropping the commit, then the parent target's push.
- way-forward 5, coverage 4, guard-reach 12 — MEDIUM — fixed as proposed: card 2 keeps the wait record across re-steps with its first start time, removes it only on a non-network stop, a done, a hand-back or a cancelled wait, and removes a stale record at driver start; card 3 reads a network-class failed child under a held driver lock as waiting, closing the gap before the first record write.
- guard-reach 5 — MEDIUM — fixed as proposed: card 6 makes every target's `Ahead` true while its entry holds a landing, so a beam pair stays on until its landings settle.
- coverage 5 — MEDIUM — fixed as proposed: card 5 adds `PairPush.SingleAttempt` and `RunPush.SingleAttempt`, set by `pusherDeps`, with a `TestPushPairAnchored` row.
- coverage 6, row-validity (card 5 auth) — MEDIUM — fixed differently: no new fabricengine row; card 5 names the existing subtest `RepeatedAuthRefusalsShareOneRecordAndOneMailAndALandedPushRemovesIt` in `internal/fabricengine/pushclasses_integration_test.go`, which pins one record and one mail on an auth refusal, beside `TestRun`'s stop row.
- coverage 7 — MEDIUM — fixed as proposed: card 11 adds `ParentTarget` and a `TestParentPush` row recording a beam landing on the beam pair's key with no landing-key entry.
- coverage 8 — MEDIUM — fixed as proposed: `TestParentPush` gains beam-parent rows for `PushTip` and the whole-push hold-back.
- coverage 9, guard-reach 9 — MEDIUM — fixed as proposed: card 10 adds the `Lym-Landing` trailer key, `LandingCommits` and `ResetParentToOrigin` (`git reset --keep origin/<parent>`, the operand D5 names, replacing `ArchiveParentMerge`), card 12 writes the trailer, and `TestParentPush` gains the unrecorded-landing and reset rows.
- coverage 10 — MEDIUM — fixed as proposed: card 10's `AheadByOnly` requires every first-parent commit past origin to be an owed tip; cards 12 and 16 use it and gain rows with an unrecorded commit below an owed tip.
- coverage 11, guard-reach 10 — MEDIUM — fixed as proposed: card 19 refuses in `EndForCancel` before any process ends, with a `TestEndForCancel` row.
- guard-reach 7, coverage 11, forced-edits (outside lens) — MEDIUM — fixed as proposed: card 19's `HandOver` skips the pair removal while the landing stands and returns `ErrHandedOver`, with a `TestPushRun_HandsOverOnATakeover` row.
- coverage 12 — MEDIUM — fixed as proposed: card 6 adds `restartHubPushers` with `TestRestartHubPushers` and `TestPruneLedger`.
- coverage 13 — MEDIUM — fixed as proposed: `TestSyncStatus` gains a refused-entry row.
- coverage 14 — MEDIUM — fixed as proposed: card 6 adds `TestPusherCmd_PushNowWakesAndExitsAtAHeldLock` and card 5 adds `TestNewDeps_WaitWakesOnAPushNowMark`.
- guard-reach 6, row-validity (card 7 red-first) — MEDIUM — fixed differently: the proposed real spawned child was not made detached; card 7's `TestSpawnStatusPush` runs the recorded command line synchronously through the built `lym` binary and asserts no failed-push record, which the merge base's `fabric push` child writes.
- guard-reach 8, coverage 20 — MEDIUM — fixed as proposed: card 13's lock-free check also runs when `PullRequest.Mode` reads `trail` or `review`, with a trail row before the lock.
- row-validity (card 17 tier0) — MEDIUM — fixed as proposed: card 17 re-signs `pushOwedForCatchUp` over `catchUpPushSeams` with `Pair` and `Held` seams.
- row-validity (card 19 `TestRemover`), coverage 17 — MEDIUM — fixed as proposed: the `TestRemover` row is cut and the tier1 `TestCommand_RemoveRefusesWhileALandingStands` asserts origin's board and records.
- row-validity (card 19 start seam) — MEDIUM — fixed as proposed: `teardownPushSeams` takes a `start` seam, re-signed with its caller `battenCLI.wire`.
- row-validity (card 19 `landingStands`) — MEDIUM — fixed as proposed: card 19 adds the `Teardown.landingStands` read wired in `New`, nil reading as none.
- row-validity, forced-edits (card 11 test package) — MEDIUM — fixed differently: the settle-only start row is gone with the settle-only pusher; the refused-start row is a row of `TestStart`, which card 11 lists.
- row-validity (card 12 parent-owner row), guard-reach 11 — MEDIUM — fixed differently: no row added; the branch is dropped, since `Deps.ParentOwner` reads the local board the reconcile left (`internal/landingshed/deps.go:159`), and card 12 says the parent-owner read keeps today's halt.
- row-validity (card 4 clock) — MEDIUM — fixed as proposed: `pushledger.Update` takes `now`, and cards 4 and 21 set the clear time through it.
- forced-edits (card 2 doc) — MEDIUM — fixed as proposed: `internal/sheddriver/doc.go` moves under Edit.
- forced-edits (card 7 docs) — MEDIUM — fixed as proposed: both package docs and the file comments are named; `internal/fabriccli/spawn.go` is an Edit target.
- forced-edits (card 19 battenshed doc) — MEDIUM — fixed as proposed: `internal/battenshed/doc.go` moves under Edit.
- forced-edits (card 12 docs) — MEDIUM — fixed as proposed: `internal/landingshed/doc.go`, `Finalize.Call` and `markTaskDone` move under Edit, and the two file comments are named in the Intent.
- coverage 15 — LOW — disputed, every fix worse: PATTERN-test-speed binds one `tiergate.Require` tier, the lowest that fits, and `decideDrive` and `goDriver` run over fakes at tier0 (`internal/sheddriver#TestDecideDrive` and `#TestGoDriver_Settle` are tier0); moving the rows to tier1 breaks that entry, which wins over the record's tier.
- coverage 16 — LOW — fixed as proposed: card 3 adds a `github-api` row.
- coverage 18 — LOW — fixed differently: no new catch-up run test; card 17 names `internal/dobbyengine#TestCatchUp`, which drives the engine that starts a run when `PushOwed` returns nil.
- coverage 19 — LOW — fixed as proposed for each branch: rows for `ReadDriveWait`'s failed read (card 3), zero `Timing` and the level pass (card 5), the board refusal (card 6), `RecordsHold` on origin (card 9), the owed prime push (card 10), a nil `ParentPushState` (card 13), a non-owed prime commit (card 16), a failed summary post (card 20), `sync: ?` (card 21) and the Bolt pull before owner reads (card 11); the parent-owner transport branch is dropped as above.
- way-forward 6 — LOW — fixed as proposed: card 13's refused halt ends with `lym loom resume`.
- row-validity (card 5 config) — LOW — fixed as proposed: a config that fails to load runs on the defaults, with a `TestPusherDeps` row.
- row-validity (card 6 `TestStart` hub) — LOW — fixed as proposed: `TestStart` moves its hub under `t.TempDir()`.
- row-validity (card 10 tier) — LOW — fixed as proposed: the `held_back` way-forward row moves to the new tier0 `TestFailedPushWayForward`.
- row-validity (card 7 repeat) — LOW — fixed as proposed: `TestSpawnStatusPush`'s command-line rows are cut, left to `TestPushSpawnCommand`.
- forced-edits (card 12 `nilLegal`) — LOW — fixed as proposed: card 12 drops the four stale `nilLegal` keys.
- forced-edits (cards 4 and 14 `lymdirs` doc) — LOW — fixed as proposed: `internal/lymdirs/doc.go` goes under Extends on both cards.
- forced-edits (card 19 runpusher doc) — LOW — fixed as proposed: `internal/runpusher/doc.go` moves under Edit.
- forced-edits (card 21 bar comment) — LOW — fixed differently: the proposed `bar.go#` target was not listed, since `redundant-file-target` refuses it beside `statusFormatSessions`; the Intent names the comment and the line-2 test goes under Uses.
- forced-edits (card 7 dead fields) — LOW — fixed as proposed: `statusPushRequest` stops setting the child-only fields and `parentLockHeldBy` is deleted; `lym fabric push` keeps reading its own flags in `teardownlock.go`.
- forced-edits (card 4 `docs/overview.md`) — LOW — fixed as proposed: `docs/overview.md` goes under Extends, `docs/overview.md:339` listing `internal/runpusher/`.
