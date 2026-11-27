```yaml
lens: existing-paths
outcome: reported
```

Lens: existing-paths. I traced the code paths that already write or read the state each decision changes.

**Findings**

1. **BLOCKING — D7.** The rule "every commit origin holds past the named tip is present in this machine's repository" is broken by fetches that already run.
   - The pair pusher runs the run-scope origin reconcile before each push (PATTERN-owner-machine). The code-tip settle in `internal/battencli/codetipsettle.go` also fetches the code and records branches.
   - Both bring origin's hand commit into the local object store, so the commit is "present" whenever `resetUnnamedTip` runs, and the reset goes through.
   - The test has to be reachability from local refs or reflog of the code branch, not presence in the object store, or the record must name how the check tells the two apart.
2. **BLOCKING — D5 against PATTERN-owner-machine.** The record does not say in what order the board target's push of `done` and the landing target's parent push reach origin.
   - If `done` lands first, the reconcile reads an entry origin ended while origin's parent lacks the work. The pair's pusher runs that reconcile before each push, and so does any resume.
   - PATTERN-owner-machine says such an entry "is paused naming what moved", so the run's own pusher and its teardown stop.
   - D5 needs the parent push to land before the board `done` is pushed, or the reconcile needs to read the local parent while a landing push is owed.
3. **MEDIUM — D5, Finalize past its merge.** `internal/pairteardown/runread.go` judges `PastParentMerge` with `originParentHolds` and falls back to `localParentHolds` only when pushes are off.
   - With the parent push owed, a resume or act-rule read after a local landing reads the run as not past its merge.
   - That defeats the "rest of a Finalize past its merge finishes on the owner" path and can re-drive the merge step.
   - The record says nothing about this read.
4. **MEDIUM — D5, landed-pair teardown.** `tearDownLeftover` in `internal/beamcli/dobbydeps.go` tears a landed pair down with `Remote: true` and checks nothing about origin's tips.
   - D5's guard ("only once origin's tracking refs hold both tips") checks the task tips but not the owed parent tip.
   - A landed pair can be removed, and its origin branches retired, while the work exists only in the local parent. A machine switch then loses it.
   - The guard should also require that the landing target holds no owed entry for that parent.
5. **MEDIUM — D1 against webster reset.** PATTERN-run-fsm says webster reset's records push is "fabric's detached push … a failure of it is owed and its record holds the run".
   - D1 turns fabric's detached pushes into pusher starts, and the pusher writes no owed record (`OwedUnrecorded`). That hold goes away.
   - The run then waits only on the code-tip wait. The local branch lacks the old tip, so the owner's dobby gives the run up once `runfsm.CodeTipGrace` (30 minutes) passes, in `settleCodeTip`.
   - An outage longer than 30 minutes, which D2's ceiling now makes normal, returns the run to its previous transition. The record does not mention this.
6. **MEDIUM — D3, ledger lifetime.** The record says when a target joins the ledger and when it leaves on a landed push, but not who clears its entry when the target disappears.
   - Targets disappear through a cancel `--remove`, a hand-over (`pairteardown.HandOver` removes the local pair), an orphan teardown, or Worktree-Teardown and the dobby's leftover teardown.
   - A stale entry keeps the summary listing a branch that no longer exists, keeps the all-clear from ever going out, and makes the `sync:` minutes grow without end (D4).
7. **MEDIUM — D3/D4, owed work no pusher has seen.** Commits made with plain `git commit` while no pusher runs are not in the ledger until the dobby's `StartIfAhead` starts one.
   - Until then the `sync:` field reads `0m` while work is only local, which is the case D4 exists to show.
   - The minutes should be derived from targets ahead of origin (`RunCommitsAhead`), or the ledger should be filled from the dobby tick's ahead check.
8. **MEDIUM — D1, board pushes that stay synchronous.** The record moves `lym board sync`'s detached push and the landing's `done` onto the board target. It is silent on the other board pushers.
   - The owner write needs its push result: `boardengine.ErrOwnerOriginUnreachable`, used by adopt and the catch-up.
   - `internal/hubreconcile/walk.go` and `internal/tendercli/deps.go` also push the board.
   - The record should say these keep their synchronous push. An owner write moved to the pusher would break PATTERN-owner-machine's "re-read after an owner write's push".
9. **MEDIUM — D6 against the dobby's stop mail.** Commit `4aa85ab3a` ("dobby: mail the parent on run stops") added a dobby path that mails on run stops.
   - D6 keeps the shed state `failed` with a transient class while the driver waits, and promises only one notice.
   - The record does not say that the dobby's stop mail and the halt classifier in `internal/loomcli/halt.go` skip a network stop the driver is waiting on. Without that, per-stop mail comes back.
10. **LOW — D2, pusher restarts.** "A start within the interval waits out the rest of it" needs the time of the last push attempt per target, which outlives a pusher process: each commit spawns a fresh `lym loom pusher`, which exits when idle.
    - D3's ledger holds only the owed-since time and the interval in force.
    - Add the last-attempt time, or a fresh pusher pushes at once on every commit.

**Paths traced, per decision**

- D1:
  - fabric's detached pushes → pusher: survives for ordinary pairs, breaks for webster reset (5).
  - Board owner write and the other board pushers: not addressed (8).
  - Prime `CodePushSkipped`: survives, since the landing target is new.
- D2:
  - Pusher started per commit: needs a stored last-attempt time (10).
  - Worktree-Teardown's `PushUntilLanded`: replaced by D5's one push, survives.
- D3:
  - Pusher exit, hand-over, cancel and teardown: stale entries (6).
  - Dobby `StartIfAhead`: the ledger misses work no pusher has seen (7).
- D4: the ledger as the only source misreads (7, 6).
- D5:
  - Run reader's `PastParentMerge`: breaks (3).
  - Pusher's run-scope reconcile, read as ended while the parent lacks the work: breaks (2).
  - Dobby `tearDownLeftover`: breaks without a parent-push check (4).
  - Code-tip settle on the owner: survives, since a held tip is never given up.
- D6:
  - `driverules.go` and `drive.go`: survive.
  - Dobby stop mail and `halt.go`: not addressed (9).
- D7: the reconcile's fetch and the code-tip settle's fetch break the presence test (1).
- D8: catch-up then pusher: survives.
- D9: `verifygate.go` with Finalize's resume re-running the gate: survives, since a removal is idempotent.

**Tool calls**

1. Grepped `CodeTipGrace`, the parent-holds readers and `TipsOnOrigin`, to find who reads the origin-dependent state D5 changes.
2. Read `codetipsettle.go` and found the parent-holds callers and the landed-pair teardown, to check the grace give-up and the teardown's guards.
3. Read `tearDownLeftover`, the run reader's seams and `OriginParentHoldsTaskWork`, to see whether the landed teardown checks origin and which parent the reader reads.
4. Grepped the uses of `originParentHolds` and `CodePushSkipped`, to find where `PastParentMerge` is judged.
5. Read the run reader's `PastParentMerge` switch, to confirm it reads origin's parent unless pushes are off.
