```yaml
lens: bound-breaker
outcome: reported
```

Lens: bound-breaker. I attacked each bound, guarantee and claim about existing code in the record.

**Claims attacked**
- D3 "a refused target leaves the ledger" with the all-clear and `sync:`: **broken**.
- D6 "a wait record under a held run lock means waiting; a free run lock means a dead driver": **broken**.
- D5 / acceptance criterion "a local landing returns done while origin is unreachable": **broken**.
- D5 "a resume after the merge never re-drives it" / "the landing target pushes the parent first and the board `done` after": **broken**.
- D6 bound "re-stepped for as long as it stays network-class": **broken** for permanent faults that look like network faults.
- D7 reachability bound (a local branch or the code branch's reflog): **broken** by the reflog case.
- D7 "fabric's push classes and the classifier read auth failures and GitHub 401 as refusals": **held**, but it is already true at the merge base, so the claim overstates the change.
- D7 "the same rule binds every other push path that resets or forces": **open**; the paths are named by reference only.
- D1/D2 "one background pusher per target": **broken** for a landing onto a beam parent.
- D8 "a held push lock is owed": **held** (`IsPushOwed` treats `ErrPushLockBusy` as owed, `pushanchored.go:66`).
- D10 "no gate package reads origin": **held**. The gatecli hits read the local fabric origin record (`gatecli/run.go:525-530`, `scope.go:728`), never the remote.
- D12 #218 settled: **held** (`pairteardown/runread.go:252-266` skips the code-tip refusal when the local pair holds the tip).
- D13 "a landed pair is removed once origin holds both tips and the parent owes no push": **held** for the pair's own work. A refused parent push leaves the ledger, but removing the pair loses nothing, since the parent branch is separate.
- D9 "ignore rules read from the first parent's tree": **broken**; git has no such mode.
- D4 "`sync-status` spawns nothing and touches no network": **held** as stated; the bar job runs at each 5 s redraw (`bar.go:201`).

**Findings**

1. **BLOCKING · design · decision-record.md D3 (with D1/D4).** A refused target leaves the ledger, so the hub reads as fully pushed while a refused branch is still only local.
   - Neither pusher start path starts a pusher while a refused-push record stands (`runpusher/start.go:122-124`, `run.go:66-68`). So the dobby tick's ahead check never writes the entry back.
   - D3 then posts "every branch is pushed", and D4 shows `sync: 0m` with commits still only local. That is exactly the state the operator checks before a machine switch.
   - If the ahead check does write the entry whenever the branch is ahead, it contradicts "leaves the ledger", and the all-clear never fires.
   - The record says neither which of the two happens nor what clears the refusal.

2. **BLOCKING · design · decision-record.md D6.** "The driver holds its run lock while it waits" conflicts with the driver's own recheck and with the step it re-steps.
   - Today the driver holds the driver lock (`battencli/drive.go:72`). The step holds the run lock (`loomcli/wiring.go:400`, `shedrun.RunLock`).
   - After each wait, `recheck` probes the run lock with `TryAcquireWriteLock` (`battencli/wire.go:193-204`, `sheddriver/drive.go:854-858`). The lock is a flock (`lock/lock.go:27`), so a second descriptor in the same process reads it as held, and the driver exits as "running elsewhere".
   - The re-stepped `lym loom step` would also meet a held run lock and stop busy.
   - If "run lock" means the driver lock, the record names the wrong lock and its dead-driver rule is unstated.

3. **BLOCKING · design · D5/D6 and the D5 acceptance criterion.** A local landing cannot return done while origin is unreachable.
   - Before the merge, Finalize runs the owner check's run-scope reconcile (`finalize.go:359`, `439`). On an unreachable origin that check halts (`finalize_owner.go:53-54`), and D6 turns it into a `git-transport` wait.
   - `handedOverStop` pulls Bolt from origin before every push step (`loomcli/landingdeps.go:173-176`, `finalize_owner.go:120-127`).
   - So the run waits instead of landing. That contradicts the Goal and the criterion "returns done while origin is unreachable".

4. **MEDIUM · design · D5 resume and ordering.**
   - A crash between `parentHandle.Merge` and the new owed record leaves no landing entry. The run reader then reads origin's parent (`runread.go:273-280`), which lacks the work, so `PastParentMerge` is false. Nothing in the record says the resumed Finalize records the owed tip again (it skips the merge, since `ParentHoldsTaskWork` reads the local parent, `landingdeps.go:165-171`).
   - The board `done` is a board write. Every board write spawns the board sync (`boardengine/board.go:216-218`), which D1 moves to the board's own target. So "the landing target pushes `done` only after the parent" is not enforced: the board target pushes `done` with any other owed board commit.

5. **MEDIUM · design · D1/D2 targets.** A landing onto a beam parent gets two pushers on one branch.
   - D1 makes the beam pair's target owe "either branch", the beam code branch included, and also adds a landing target for that same parent branch.
   - The pusher lock is per records worktree (`runpusher/target.go:48-51`), so the two targets either share the beam pair's lock or race each other unordered.
   - Both then break "one pusher per target" and the parent-before-`done` order. D3's ledger would also list the branch twice.

6. **MEDIUM · design · D9.** git cannot evaluate ignore rules from a tree.
   - `git check-ignore` and `git ls-files` read per-directory ignore files from the worktree. Their only tree option is `--with-tree` (`git check-ignore -h` on git 2.53.0), and it does not read ignore rules from the tree.
   - So the decision needs lym to read the ignore blobs and match them itself, which leaves nested ignore files, negations and `info/exclude` unsettled and unpinned.

7. **LOW · design · D7 reflog bound.**
   - A hand commit can enter the local code branch's reflog without the branch keeping it, for example `git reset --hard origin/&lt;branch&gt;` followed by a reset back. It then passes the reachability test, and the reset drops it.
   - The other way, lym's own rewound commits fall out of the reflog after expiry (`gc.reflogExpireUnreachable`, 30 days by default). The reset is then refused, and the record names no way forward beyond the takeover case.
   - None of the git behaviour relied on is pinned by a test, a recorded command or the docs.

8. **LOW · design · D6 bound.** Permanent faults can read as network-class and be re-stepped forever.
   - A remote URL that never resolves ("could not resolve host", `gitexec/transport.go:27`) or a GitHub API proxy that refuses every connection both classify as transient: `githubclient.IsTransient` returns true for any non-parse `url.Error` (`githubclient/transient.go:43-47`).
   - The stated bound gives no way out except `lym batten cancel` and the summary.

9. **LOW · consistency · D7 classifier claim.**
   - `authRefusalTriggers` already holds "Authentication failed" and "could not read Username" (`gitrepo/push.go:58`), and `rejectionRetryKind` treats `ErrPushAuthRefused` as final (`pushclassified.go:345`).
   - A GitHub 401 is already not transient (`githubclient/transient.go:40-41`).
   - So D7's classifier sentence and its auth-refusal criterion describe existing behaviour as new work.

10. **LOW · design · D7 "every other push path that resets or forces".**
    - `updateRemoteBranch` has two other callers that move an origin branch: `fabricengine/handover.go:274` and `destroy.go:2697`.
    - The record binds them only through the phrase "every other push path", and only through `pr-modes-no-agent`'s list. None of the five paths that list names forces a push: they reach the reset only through `resetUnnamedTip`.
    - So whether the rule binds those two callers is left open.

**Tool calls**
1. `gitexec/transport.go` phrases, the `gitrepo` auth error, `runLockHeld`, and a grep for origin reads in the gate packages: to test D6, D7 and D10, which the manifest held only by name.
2. The gatecli origin hits, `authRefusalTriggers`, and the driver-lock wiring: to settle the D10 hits, D7's auth claim and which lock the driver holds.
3. The `internal/lock` implementation, force-update sites, and the `ParentHoldsTaskWork` wiring: to see whether the lock is per descriptor, and how D5 behaves on a resume.
4. The `updateRemoteBranch` callers and the `OwnedElsewhere` wiring: for D7's other force paths and D5's origin reads.
