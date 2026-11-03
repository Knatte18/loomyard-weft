# Lens review — batten-remote-order plan

Every lens fork returned a report; each is saved beside this file as `lens-review.lens-<lens>.md`.

## coverage

- coverage 1, MEDIUM — fixed as proposed: card 9 gains a row whose origin Bolt reads the entry done; the entry reads ended with `PairAbsent` and a recording `Teardown` seam is never called.
- coverage 2, MEDIUM — fixed as proposed: card 9's `wanted` filter now matches no warp worktree under the absent case, whatever `ReadOriginFor` finds, and its row keeps the records worktree standing with an origin record naming the prime's branch and asserts no outcome for that branch.
- coverage 3, MEDIUM (merged with row-validity 5) — fixed as proposed: cards 10 and 12 gain tier0 rows where `Add` returns a `create worktree … failed` error, read as transient with the status still `running`.
- coverage 4, MEDIUM — fixed as proposed: card 6 gains a row with origin's weft tip ahead of the local tip: no branch push, a tag at each tip, `OriginArchiveTag` at origin's tip.
- coverage 5, MEDIUM — disputed, every fix worse: the failed-push way forward is built inside fabricengine (`internal/fabricengine/failedpush.go:215` `FailedPushWayForward`), which `PATTERN-fabric-isolation` bars from naming batten, and a hook letting batten append to it is "a seam only one module could use", which the same entry forbids; the prompt's constraint rule records the conflict in Shared Decisions (`d3-sites-outside-fabric`, rationale extended with this), and card 13's `lym fabric remove` warning reaches the operator after the removal that record names.
- coverage 6, MEDIUM — disputed, every fix worse: a test of the rule's wording pins a stencil's static prose, which `PATTERN-test-speed` forbids ("No test pins a stencil's static prose"); the rule adds no marker or Go-filled value, and Shared Decision `d5-no-prose-test` records the conflict.
- coverage 7, LOW — fixed as proposed: card 12 lists `internal/dobbyengine#TestPlanPass` and adds a `failed` row that plans no pass.
- coverage 8, LOW — fixed as proposed: card 10's integration test gains a beam project's pair at a subpath anchor re-made at the anchor its weft records.
- coverage 9, LOW — fixed as proposed: card 10's tier0 table gains rows for a first-reconcile error and a prune error.
- coverage 10, LOW — fixed as proposed: card 4 gains rows for a weft absent locally and on origin, an adopted local warp branch behind origin's copy kept at its own tip with no tracking ref written, and an adopted weft with no origin record given no record commit.
- coverage 11, LOW — fixed as proposed: card 5 gains rows for the discard's wait-out refusal, `PushAnchored`'s skip under a held lock, and the prime's records push going out while a pair's lock is held.
- coverage 12, LOW — fixed as proposed: card 3 gains a row whose planted `config.lock` fails the drop before any worktree is made, and its first row asserts the weft's upstream config unset too.
- coverage 13, LOW — fixed differently: proposed an assertion for both the code-branch and pause-gate parts; card 6's first row now asserts the code bare's task branch unchanged, and the pause-gate part takes no row because card 2 states `PushBranch` adds no gate branch, reaching the gate through the same `pushPlainWithRetry` path `PushRebaseFree` takes, so there is no new branch to reach.
- coverage 14, LOW — fixed as proposed: card 7 gains a removal row where origin holds no weft branch and the branch push is refused: no remote delete is attempted and nothing is reported.
- coverage 15, LOW — fixed as proposed: card 8 gains a row where a file at the teardown lock's directory fails the lock with an error other than busy; the entry carries `Error` and the branch stands.
- coverage 16, LOW — fixed as proposed: card 11 gains an ls-remote failure row; card 12 gains a candidate with no status file (skipped) and a `Tags` error (transient refusal, status untouched, no notice), and states that behaviour in its Intent.
- coverage 17, LOW (merged with row-validity 6) — fixed differently: proposed rows through `Result.Warning`; the `Run` rows (Run-Shed, Land, owner-held, `LeaveClaim`) move to the existing `TestTeardown_CallSequence` over `fakeSubstrates`, which card 13 now lists, and `TestBattenFollowUp` keeps the wording table only.

## row-validity

- row-validity 1, MEDIUM — fixed differently: proposed making the delays or the sleep injectable like the hook; a package-level sleep would break `PATTERN-test-speed`'s "a struct field or a parameter, never a package-level var", so card 6 adds `Topology.archiveSleep`, threaded as a `sleep` parameter through `archiveWeftTip`, `archiveTaskTip` and `tagAndPushArchiveTip`, with `SetArchiveSleepForTest` beside `SetTeardownWaitForTest`.
- row-validity 2, MEDIUM — fixed as proposed: card 6's diverged-tip row removes with `remote` false, so only the second archive tag makes the later `Add` read the pair not live.
- row-validity 3, MEDIUM — fixed as proposed: card 9's rows call `fabricengine.ReconcileOrigin` directly and list `publishBoard` and `originBoard` under Uses.
- row-validity 4, MEDIUM — fixed differently: proposed `squashLand` and `pushedPair`, which are `fabricengine_test` helpers a battencli test cannot import; card 12's Land row squash-lands the task work on the parent and pushes it itself, then removes with `remote` true.
- row-validity 5, MEDIUM — fixed differently: proposed a hook between the reconcile and `Add`; card 12 drops the integration row instead, since the transient answers at `Add` are reached by the tier0 rows of cards 10 and 12 over a fake `Add`, and a hook seam in the pre-run would exist only for the test.
- row-validity 6, MEDIUM — fixed as proposed: see coverage 17; card 13 lists `fakeSubstrates` and puts the `Run` rows in `TestTeardown_CallSequence`.
- row-validity 7, LOW (merged with way-forward 4) — fixed differently: proposed saying the rows install a recording `CommitStatus`; card 12 now commits through its own `readopt.CommitFailed` seam, and the rows install a recording one.
- row-validity 8, LOW — fixed as proposed: `readoptSeams` gains `RecordsRoot`, faked at tier0.
- row-validity 9, LOW — card 7's no-hook row: fixed differently: proposed an added lease assertion in `TestRemove_RemoteTrueDeletesWeftBranchOnRemote`; the row is dropped, since that test runs the leased path once card 7 lands and the lease itself is not observable on the result. Card 10's prime-dirt and stale-registration rows: fixed as proposed, dropped, as card 4's rows reach them. Card 4's stale-tracking row: disputed, premise false: card 10's D8 row asserts the second reconcile's verdict after a batten re-adopt, which card 4's fabric-level row never reaches, and card 4's row covers the behaviour card 4 introduces.

## forced-edits

- forced-edits 1, BLOCKING — fixed as proposed: card 6 lists `ArchiveWeftTipForTest` and its callers `TestArchiveWeftTip`, `TestArchiveWeftTip_NoOriginSkips`, `TestArchiveWeftTip_UnreachableOriginErrors` and `TestOriginWrites_LocalMarkedBranch`, with their new call shape and the stale comment.
- forced-edits 2, BLOCKING — fixed differently: proposed making existing continuable rows build a resolving task worktree, which a tier0 test cannot spawn; `refuseContinuePass` reads `readopt.Gone` instead, a nil seam checking nothing, so the existing rows stand and the new row sets the fake.
- forced-edits 3, MEDIUM — fixed as proposed: card 2 re-signs `Repo.pushPlainWithRetry` with `remote, branch`, both empty keeping today's push, and lists `PushRebaseFree`, its one caller.
- forced-edits 4, MEDIUM — fixed as proposed: card 13 lists `internal/pairteardown#New` and `internal/tendercli#EndOwnedElsewhere`.
- forced-edits 5, LOW — fixed as proposed: card 1 lists `Repo.PushRebaseFree` for its doc comment.
- forced-edits 6, LOW — fixed as proposed: cards 3, 4 and 5 list `Topology.Add` and name its doc-comment change.
- forced-edits 7, LOW — fixed as proposed: card 5 lists `TeardownLockPath` and `ErrPairPushRunning` and names the file-header comment of `teardownlock.go`.
- forced-edits 8, LOW — fixed differently: proposed stating the message change; card 1 instead wraps the sentinel with today's text, `gitrepo: git push --delete (leased at %s): %w`, so no caller's message changes.

## guard-reach

- guard-reach 1, MEDIUM — fixed as proposed: card 10 lists `StartFirstPass`, which sets `readopt` on its receiver, and `TestStartFirstPass` asserts it.
- guard-reach 2, LOW — fixed as proposed: card 7 orders the empty-lease check after `removeWeftWorktree`'s no-origin check.
- guard-reach 3, LOW — fixed as proposed: card 5's Intent says the synchronous pushes skip under any exclusive holder, the commits staying local for the next push.
- guard-reach 4, LOW — fixed as proposed: card 8 adds the per-slug try to `Topology.cleanupRemoteWarp`, under the Goal and the record's "cleanup sweep" scope, with a row in `TestCleanupRemoteWarp_ApplyDeletesOnlyTheCandidate`.
- guard-reach 5, LOW — fixed as proposed: card 12 asks `SlugRecordsTags` for both the `<slug>` and the legacy `self` status paths, with a row for a `done` candidate under `self`.

## way-forward

- way-forward 1, LOW — fixed differently: proposed a caller-passed rerun; `DiscardTaskBranch` is exported with callers outside fabric, and a generic `re-run the command that discarded the task branch once it finishes` names the step without re-signing it or naming a batten verb in fabric.
- way-forward 2, LOW — fixed as proposed: card 8's skip names `re-run the cleanup once it finishes`, no verb.
- way-forward 3, LOW — fixed differently: proposed a marker key of its own; card 12 posts through `notifyRunParent`, which carries no hand-back marker.
- way-forward 4, LOW — fixed differently: see row-validity 7; `readopt.CommitFailed` commits through `battenCommitStatusDeps` and spawns the push, bypassing the no-op marker.
- way-forward 5, LOW — fixed as proposed: the tag step is skipped when the tag already stands at `<tip>`.
- way-forward 6, LOW — fixed as proposed: with no `done` candidate the stop names finishing by hand and no task-branch step, with a row asserting it.
- way-forward 7, LOW — disputed, premise false: `deny_publish` is set only by the sampler's and PR-review session's launch specs (`internal/samplerengine/spec.go:102`, `internal/landingshed/prsession.go:147`), never the orch parent's, so the named `git push` steps are not denied to the receiver; card 12 now states this.
