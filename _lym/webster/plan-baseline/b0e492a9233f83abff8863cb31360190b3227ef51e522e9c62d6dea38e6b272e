# Card 17 — hand-over-owned-elsewhere

**Create:**
- `internal/pairteardown#HandOverOwnedElsewhere`
- `internal/pairteardown_test#TestHandOverOwnedElsewhere`

**Extends:**
- `internal/pairteardown/doc.go`

**Uses:**
- `internal/pairteardown#HandOver`
- `internal/pairteardown#PushRun`
- `internal/pairteardown#RunPush`
- `internal/pairteardown_test#TestPushRun_HandsOverOnATakeover`

**Intent:** Add the catch-up's step 4: hand over a run another machine owns, from this machine's side (D4 step 4).

`HandOverOwnedElsewhere(prime, slug, owner)`:
1. reads the run's batten record on origin (`fabricengine.OriginBattenStatus`); when it holds no takeover entry, it first pushes the run's owed commits, records side first, through `PushRun` over the prime's records and then over the run's pair when it stands here (`RunPush.LockWait` `fabricengine.StatusPushLockWait`), so this machine's transitions reach origin before it lets go;
   a push that meets a takeover entry hands over inside `PushRun` and returns `ErrHandedOver`, which ends the call with nil;
2. then calls `HandOver(prime, slug, shedengine.Takeover{Machine: owner}, RunPush{})`, which ends this machine's processes for the slug through `EndSubjectProcesses` with no dobby end, pushes the code tip the run's last transition names when origin lacks it, archives the local tips and pushes the archive tags, posts the one notice naming the machine and any dropped pause or cancel intent, writes the hand-over mark under the hub's `.lym` once the archive lands, and removes the local pair through local mode, which takes no origin step and writes no board;
3. answers nil for the `ErrHandedOver` that `HandOver` returns, and any other error naming the run and the step.
A `landing` run past its parent-side merge is handed over the same way, so its owner finishes the Finalize.
A failed push for a transient reason returns the push's error, so the catch-up retries; nothing is archived first.

Bound: it hands over only a run whose owner reads elsewhere after the catch-up's registration landed (card 18 step 1), so a refused or unregistered identity never reaches it.
The local pair's forced removal discards uncommitted files only after the archive tags naming its tips are on origin, since `HandOver` returns the archive's error before it removes the pair.
It pushes only commits origin lacks, never rewinds or force-pushes an origin branch, writes no board and touches no other machine's records.

`TestHandOverOwnedElsewhere`, tier1, over a hub fixture with a local pair whose records side holds a commit origin lacks, a process registered under the slug in `internal/runproc` and a local pause intent origin lacks:
the commit reaches origin, the archive tags reach origin, the registered process is ended, the notice names the owner and the dropped pause intent (read as `TestPushRun_HandsOverOnATakeover` reads it), the local pair is gone and origin's branches stand, and the hand-over mark names the owner;
a run at Finalize past its parent-side merge is handed over the same way, its process ended;
a second run whose origin record holds a takeover entry pushes nothing first and answers nil;
with the pair's origin unreachable, the call returns the push's error and writes no archive tag and no mark.

**ImpactSummary:** New exported function composing PushRun and HandOver; no existing caller changes.
