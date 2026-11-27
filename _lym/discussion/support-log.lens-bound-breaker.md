```yaml
lens: bound-breaker
outcome: reported
```

Bound-breaker lens: I attacked the record's claims, bounds and ordering promises with concrete counterexamples.

**BLOCKING — D7: the "present in this machine's repository" test does not identify a hand commit.**
Every fetch of origin copies origin's objects into the local store: the pusher's origin reconcile before each push, the catch-up fetch, and the push-classified path that reads origin's tip.
So a commit made by hand on GitHub is present locally by the time `resetUnnamedTip` runs, the test passes, and the reset drops it, which is exactly the fault the record means to stop.
A test that could work: "reachable from a local branch or the reflog of the code branch", never "present in the object store".
D7's acceptance criterion, "origin holds a commit this machine lacks", only tests the never-fetched case, so it would pass while the bug stays.

**BLOCKING — D6: the dobby's run watch still sends one mail per re-step.**
Commit `4aa85ab3a` added a run watch to the dobby. On each tick it mails one `[run watch]` line for every stop, meaning any reading whose state is not running, the child's shed state included. It uses an episode stamp to tell a second stop at the same row from the first.
D6 keeps the shed state `failed` with its transient class between re-steps. Each re-step that fails again is therefore a new episode and a new mail, so per-failure mail comes back for a run waiting at Finalize.
The claims "no halt note", "one parent notice" and "`lym loom status` reads it as waiting" do not cover this reader. The record never mentions the run watch.

**MEDIUM — D2: "a start within the interval waits out the rest" has no state to read.**
D3's ledger holds only owed targets, and a target leaves it when its push lands.
Sequence: a push lands, the entry goes, and a commit 5 s later starts a new pusher. Nothing records when the last push happened, so it pushes at once.
The interval is then kept only within one pusher process's lifetime, never across starts, which defeats "fewer pushes".
The last push time per target has to be persisted outside the owed set.

**MEDIUM — D3/D4: work that is ahead but has no pusher never enters the ledger.**
An agent's plain `git commit` starts no pusher; issue #247 is exactly this case. Until the dobby tick's `StartIfAhead` runs, and on a branch with no dobby at all, the ledger is empty and the bar reads `sync: 0m` while the work is only local.
This breaks D4's stated purpose, showing before a machine switch whether work is still only local.
"Owed" needs to mean "ahead of origin", not "has a pusher retrying".

**MEDIUM — D3: the shared "last had no owed target" time races.**
The record keeps one file per target plus one hub-wide time.
Interleaving: pusher A removes its entry and sees the ledger empty; meanwhile pusher B adds its own entry; A then writes "last clear = now".
The bar now undercounts B's owed period, and the all-clear check can fire too early. No lock is named.

**MEDIUM — D9: the pre-merge ignored list does not survive a stop between the merge and the clean check.**
Finalize can halt or crash after the merge-in commit and before the clean-tree check; that is the very halt issue #242 shows. The resume then has no pre-merge list, because the merge is already done.
The leftovers then halt every resume forever. The list has to be derivable after the merge, for example from the merge's first parent tree's ignore rules, or persisted with the step.

**MEDIUM — D5: a PR landing followed by a local landing on the same parent hits the refusal path.**
GitHub merges into origin's parent, so the local parent lacks that merge commit.
A later local landing merges onto the stale local parent, its pusher push is rejected, and D7 turns that into an immediate refusal and stop.
D5 orders only "PR landing waits for the owed local push", never the reverse, which would be "a local landing first brings the local parent up to origin's".

**MEDIUM — D5: the claim about the dobby's landed-pair teardown is new behaviour stated as existing.**
The record says the dobby "removes the pair only once origin's tracking refs hold both tips". Nothing cited shows that pairteardown's landed-pair route checks tips today.
The tracking refs move only when a push lands or a fetch runs, so the check also needs the pusher to keep running for a landed pair.
The run-scope reconcile lets an ended entry through only when "the parent already holds the run's work", and that holds after a landing. The check itself is still a new mechanism and needs its own decision line and test; the D5 criterion covers it only in part.

**LOW — D6: the scope of `ErrOwnerOriginUnreachable`.**
The error is also returned to `battencli/catchup.go` and `adoptcli/adopt.go`.
Giving it the class `git-transport` changes those callers too, while Scope says the catch-up's fetch failure stays as it is. The record does not mention adopt.

**LOW — D7: "a rejected push is a refusal" overstates today's code.**
A rejected records push first reads origin for another machine's takeover and hands the run over. The pusher also retries a rejection rebase-free.
Calling every rejection a refusal that "stops at once" contradicts those two paths unless they are named as exceptions.

**LOW — D3: losing the pusher's ceiling notice leaves no signal when the prime's dobby is down.**
The auto-mode assumption that "the prime's dobby runs whenever the hub has work" fails for a hub with only board or landing pushes owed and no run in flight.
The pusher's own notice is removed, so nobody hears of the outage.

Claims I attacked that held:
- D5: the parent lock is released before the push, and a second local landing stacks on the first. The pusher's fast-forward carries both.
- D6: the wait runs between steps, so no parent lock is held during it; Finalize's lock is released when the step returns.
- D8: starting a run with its push still owed is within PATTERN-run-fsm's accepted window ("both machines may run rows whose acts stay local").
- D12 (#218): with the pair standing here, the code-tip wait does not hold the run.
- D1: the prime's code branch is pushed fast-forward only, and a push the operator moved on origin is refused, which matches the operator's refusal rule.

Tool calls: one. I read the message of `4aa85ab3a` and grepped `dobbyengine` for failed states, to learn whether the dobby mails on run stops. My context had only the commit title.
