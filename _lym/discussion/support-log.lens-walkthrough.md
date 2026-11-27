```yaml
lens: walkthrough
outcome: reported
```

Walkthrough lens: I ran each decision's mechanism end to end on one concrete case and checked whether it meets its own purpose.

**Findings**

1. **BLOCKING, D7.** The hand-commit guard never fires.
   - Case: someone adds hand commit H on GitHub to the task's code branch, and the pair is pushed. The push is rejected, `repushOverOriginContained` fetches, and `resetUnnamedTip` runs.
   - That code fetches the code side before it reads origin's tip (`pushclassified.go`, the `j.repo.Fetch()` calls, then `UpstreamSHA` and `isAncestorOrEqual`). So by the time of the check, H is already in this machine's object store.
   - D7 says to reset "only when every commit origin holds past the named tip is present in this machine's repository". H passes that test, so the reset drops H and the purpose fails.
   - Fix: the test has to be reachability from a local branch or that branch's reflog, with remote-tracking refs and their reflogs excluded. The support log's Q7 and its rejected alternative rest on the same false premise.

2. **MEDIUM, D1 / Goal (issue #247).** The record never says what the pusher's "ahead" read covers.
   - Case: an agent runs plain `git commit` on the code branch after the last transition.
   - `RunCommitsAhead` (`pusherreads.go`) first checks whether the records side is ahead. Otherwise it reads the code side through the records tip's `Warp-SHA` trailer, the tip named by the last transition.
   - So a code commit past the named tip may never count as owed, and `resetUnnamedTip`'s rule treats commits past the named tip as droppable.
   - The same holds for a landing's merge commit on a beam branch. The beam pair has no transition naming that commit, so D1's line "the pair target covers it" does not follow.
   - D1 needs to state the ahead rule for code commits made outside a transition, and for a beam parent's landing tip.

3. **MEDIUM, D2 with D3.** "A start within the interval waits out the rest of it" needs each target's last push time, but nothing keeps it.
   - Case: a push lands at 10:00. The pusher drops the target from the ledger and exits. A commit at 10:00:20 starts a new pusher.
   - That pusher has no last-push time to measure the interval from, because D3 keeps a target in the ledger only while it is owed.
   - The record also never defines "owed": ahead of origin, or ahead after a failed push. That definition decides what D4's minutes and D3's threshold count.

4. **MEDIUM, D5 with D2.** "Pushes at once" can lose to a pusher that is already running.
   - Case: the root landing's target runs under the prime's records-worktree pusher. That pusher holds its lock, for example in a 300 s owed backoff or a normal interval wait.
   - The landing's `Start` finds the lock held and exits. The parent push then waits for the running pusher's next wake.
   - D5 needs a way to wake the running pusher, or the landing target needs its own lock.

5. **MEDIUM, D5.** It doesn't say where the pull-request landing's wait runs.
   - The record says it "waits until its parent's owed landing push has landed", but not where.
   - If the wait is inside Finalize's step, it waits under the parent lock, which the support log's rejected alternative rules out. It is only safe as a driver-level wait or a transient stop under D6. The record should say which.

6. **MEDIUM, D6.** A driver that dies during its wait leaves a run that looks like it is waiting forever.
   - Case: a network stop, then the driver waits 300 s, and the machine reboots.
   - The status file reads `failed` plus `git-transport`. No halt note was written, and `lym loom status` reads that state as "waiting".
   - Nothing then tells a live wait from a dead driver, so no one resumes the run. That is issue #254's silent stall in a new shape.
   - The record needs a live-driver marker, such as the run lock or a wait record, to tell the two apart, and should say what resumes the run.

7. **MEDIUM, D9.** A re-run Finalize has no list from before its merge-in.
   - Case: issue #242's own state. A run is already blocked after its merge-in, or a step crashes between the merge and the clean check. On resume, Finalize does not merge again.
   - So the "ignored before the merge-in" list either does not exist or comes from the tree after the merge, and the run still halts.
   - D9 should persist the list with the merge-in, or derive it on resume from the merge's base tree with `git check-ignore` against the parent's tree.

8. **LOW, D6.** Adding the `git-transport` class to `boardengine.ErrOwnerOriginUnreachable` also reaches its other callers.
   - Those callers are `battencli/catchup.go` and `adoptcli/adopt.go`.
   - That can turn a catch-up fetch failure into a re-stepped transient stop, which the record's Scope rules out ("A fetch failure in the dobby's catch-up ... this task keeps it").

9. **LOW, D4.** It doesn't say how the `#()` job finds its hub.
   - The ledger is per hub, but the job runs in tmux's environment. The record does not say how `lym fabric sync-status` learns the hub directory, for example a server-global option set when the session starts, as the Slack job's command option is.

**Decisions walked**

| Decision | Case | Purpose |
|---|---|---|
| D1 | plain `git commit` on a pair, and a beam landing | not met (finding 2) |
| D2 | a landed push, then a commit 20 s later | not met (finding 3) |
| D3 | two targets owed past 30 min, then both land | met; the "last empty" time needs one writer when pushers race, which the record does not name |
| D4 | job reads the ledger | met apart from finding 9 |
| D5 | root landing during the prime pusher's interval wait; pull-request landing with its parent owed | partly met (findings 4 and 5) |
| D6 | 40 min outage at Finalize | met while the driver lives; not met after a driver death (finding 6) |
| D7 | GitHub hand commit | not met (finding 1) |
| D8 | catch-up with the push lock held by the running pusher | met: a second pusher start exits harmlessly and the run starts |
| D9 | first pass met; resume after an earlier block | not met (finding 7) |
| D10 | stencil wording | met |
| D11 | tests | met |
| D12 | issues #218 and #254 | met |

**Tool calls**

- Read `RunCommitsAhead` and the fetch and lease lines around `resetUnnamedTip`. My context had only the explorer's summary, without the ahead rule or the fetch before the reset.
