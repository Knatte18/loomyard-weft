# Retry pushes quietly and land without waiting on a push

A push to origin or a GitHub call that failed on a flaky network stopped runs and mailed the orch once per failure.
Now one background pusher per target brings what lym owns to origin and retries quietly,
a local landing hands its push to that pusher, and the Go driver waits out network stops.
A refusal that retrying cannot fix still stops at once.

## What changes

**Pushing**

- `internal/runpusher` serves these targets: the prime's records, a task or beam pair, the board's branch,
  and the prime's code branch when a landing owes it a push.
- The detached push spawns (`SpawnDetachedPush`, `SpawnStatusPush`, `boardengine.SpawnSync`) start or wake
  the target's pusher instead of pushing themselves.
- A pair is owed while either branch is ahead of origin, a plain `git commit` included.
  The prime's code branch is pushed only to a tip a landing recorded, fast-forward only.
- A target is pushed at most once per interval. The interval doubles on failure up to a ceiling and resets on success.
  A push-now mark, set by a landing and by Worktree-Teardown among others, skips the wait once,
  so a landing is pushed at once.
  New `fabric.yaml` machine keys: `push_interval_s` (60), `push_ceiling_s` (300), `push_notice_after_min` (30).
- Owed pushes are kept in a machine-local ledger (new `internal/pushledger`).
  A transport failure in the background pusher or the catch-up writes no failed-push record and sends no mail.
  Synchronous pushes such as `webster reset`'s still record one.
  Past the threshold the prime's dobby posts one summary of owed branches and waiting runs, then one all-clear.
- Every hub session bar shows `sync: <minutes>m · <seconds>s`, read by the hidden `lym fabric sync-status`.
- Both engines' edit directives say a failed push needs no action.

**Landing**

- Finalize pushes nothing. Once origin answers its owner checks, it merges under the parent lock,
  records the landing on the parent's push target and returns Done.
  That pusher pushes the parent, writes the board `done` once origin holds the work, and then releases the run's held records.
  With pushes off, Finalize writes `done` itself.
- Every local landing leaves a commit on the parent with a trailer naming the run; a non-squash landing no longer fast-forwards.
- A pull-request landing waits while its parent owes a landing push, and halts when that push is refused.
- Worktree-Teardown pushes nothing. A pair that is not yet level is left standing for the dobby to remove.
- While a run's landing is unpushed, every teardown of its pair refuses, `lym fabric remove --force` included.
- Finalize's merge-in removes ignored files left in directories the merge deleted.

**Waiting and refusing**

- The Go driver re-steps `git-transport` and `github-api` stops without a cap, from 15 s doubling to the push ceiling.
  A waiting run writes no halt note and shows as waiting.
  Finalize's owner check and Land's pull of the prime now report an unreachable origin as `git-transport`.
- The dobby's catch-up leaves an owed push, a busy push lock included, to the pusher and starts the run.
- A rejected code push no longer resets origin past a commit that neither a local branch
  nor the reflog of the pushed code branch reaches. It is refused, naming the commit.
- `webster reset` pushes the run's records itself.
  A failure is reported in `push_error` and recorded as a failed push, which holds the run.
- A pause lift pushes each pair it uncovers that holds commits origin lacks.
  A pair still covered by a live pause, owned by another machine or gone is not pushed.

Closes #247
Closes #238
Closes #243
Closes #240
Closes #218
Closes #242
Closes #254

#218 and #254 were already settled by the asynchronous run pusher (`async-run-pusher`, `b4236816a`),
and no code changes for them here.
A run no longer waits on its pair's push of the code tip (#218),
and transitions no longer push, so the status-push loop #254 reports is gone.
The bound #254 asked for on the owed retry is replaced by the quiet unbounded retry,
with the summary notice and the `sync:` field making an outage visible.

## Design decisions

- **Synchronous pushes remain where the caller reads the result:** the board owner write, `lym config`'s hub-wide commit,
  `webster reset`, the dobby's catch-up and PR-Review's task-branch push.
- **Records hold.** No push of a run's pair carries the records commit its unpushed landing holds, or any later one,
  and the board `done` waits for the parent push,
  so origin reads neither the pair's records nor the board entry as ended while the parent lacks the work.
  Batten's run record, in the prime's records, is not held.
- **Owners are read before pushing.** The parent's pusher reads the owner of the parent and of every run a push carries,
  and the run's owner again before its `done` write.
  Another machine's ownership is a refusal naming `lym adopt`.
- **Way forward per parent:** `git push` in the prime's worktree for the prime's code branch,
  `lym fabric push` for a beam parent.
- **Work already on origin under another commit** drops the local landing instead of pushing it twice.
  The local parent is reset when that landing commit alone puts it ahead; otherwise the push is refused, naming a rebase.
- **Leftover removal** judges files by the per-directory `.gitignore` files of the pre-merge head only,
  using `git check-ignore` in a scratch repository.
  Every untracked regular file they ignore under an emptied directory is deleted,
  ignored local settings or secrets included. Symlinks and nested repositories stay.
- **Dead code** left without a production caller is removed together with its tests.

## Known limits

- A catch-up whose fetch fails on an unreachable origin still stops that run's start.
- An unpushed landing holds every later pull-request landing on the same parent, and the summary notice is its only signal.
- A misconfigured origin host or a proxy that refuses every connection is waited on like an outage,
  until `lym batten cancel`.
- The reset guard relies on git's reflog expiry.
  A hand commit that passed through the local branch's reflog without being kept can still be reset away,
  and after a takeover the new owner's reset is refused.

## Check by hand

Each item names a defect and where to look; confirm the change does not have it.

- Network waits of batten's own driver being invisible: `internal/battencli/drive.go`, `dobbyreads.go`.
- The board pusher never committing uncommitted board writes: `Board.PusherSync` in `internal/boardengine/sync.go`.
- A catch-up test sleeping the production push backoff: `catchUpPushSeams.BackoffSleep` in `internal/battencli/catchup.go`.
- A `done` write judged on a board not pulled from origin: the owner refresh in `internal/runpusher/parent.go`.
- A dropped landing's refused-push record never being cleared: `clearUnlessDropped` and `SettleRefused`, same file.
- A non-squash landing leaving no trailer commit: the merge options in `internal/landingshed/finalize.go`.
- Required behaviours lacking tests (pushes-off landing, `sync:` field, refusal read, a pair's network wait):
  the tests in `internal/landingshed`, `internal/fabriccli`, `internal/beamcli` and `internal/battencli`.
- A diverged prime holding an owed tip being pulled as level: `internal/fabricengine/pull.go`.
- A Finalize resume releasing the held records commit: `internal/loomcli/landingdeps.go`.
- A hub pause lift pushing no held pair: `internal/tenderengine/resume.go` and `internal/tendercli`.
- Parent-pusher behaviours lacking tests: `internal/loomcli/parentpush_integration_test.go`.
- A protected parent branch's refusal naming no usable way forward: `protectedParentWayForward`
  in `internal/fabricengine/pushclassified.go`.
- A landing commit rebased away from the parent still being pushed: `removedFromParent` in `internal/runpusher/parent.go`.
- `webster reset`'s records push leaving no owed record when it fails: `internal/webstercli/reset.go`.

## Merge danger

**Door:** one-way
**Blast radius:** hub
