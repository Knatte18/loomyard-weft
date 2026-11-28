---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    origin: lens:existing-paths
    location: _lym/discussion/decision-record.md:77
    summary: "A batched member skips every check today's Finalize runs on that run under the lock: the run-scope reconcile (ended, stopped, owned elsewhere, hand-over), the beam land check and the hand-over stop before the record; D6 runs today's owner checks for the leader alone, so a member another machine took over, or one origin ended or stopped while it waited, lands on the local parent against PATTERN-owner-machine."
    fix: "In D6, after the lock re-read and before the move, have the leader run for each member, with that member's own deps, the checks `landUnderLock` runs for its own run (`ownerChecks`' reconcile verdict and owner read, `landCheck`, and `handedOverStop`'s owner read). A member that fails one is left out: the leader rebuilds the chain without it and gates again, and gives the member no outcome, so its own Finalize meets the same check under the lock and stops as today. Bound the rebuilds as F16 asks. Add to Constraints, under PATTERN-owner-machine, that a member's ownership is judged under the lock before the move that lands it. Add an acceptance criterion: a member whose entry reads as owned elsewhere, or ended on origin, after it joined is not moved onto the parent; the other members land. Tier 1, `internal/landingshed`."
  - id: F2
    severity: BLOCKING
    class: design
    origin: handler
    location: _lym/discussion/decision-record.md:32
    summary: "D1 has a run decide whether it lands through a pull request before it joins the queue, with no lock held, and a member the leader lands never decides again under the lock; the declared PATTERN-landing text allows deciding on a pull request only in a run's Finalize under the parent lock, and today's under-lock `pullRequestLanding` is what sees a pull-request record another run wrote while this one waited."
    fix: "Change D1 so the pre-join read is only the on-disk read that today runs before the lock (`needsGitHubOnDisk`), and add to D6 that the leader runs landingshed's resolver for each member under the lock before the move, with the member's own pull-request seams. A member the resolver now sends through a pull request is left out of the chain, and gets no outcome, so its own Finalize lands it through today's path. Rewrite D10 to say a pull-request landing is decided under the lock for every member, and keep its GitHub-rules bound. Add an acceptance criterion: a member whose pull-request record appears after it joined is left out of the batch and lands alone through its pull request. Tier 1, `internal/landingshed`. The resolver's need of member-specific seams, such as the pull-request record path and the beam mode lookup from the member's worktree, is unverified: name in D2 whatever the entry must carry for the leader to build them."
  - id: F3
    severity: BLOCKING
    class: design
    origin: handler
    location: _lym/discussion/decision-record.md:82
    summary: "D8 routes a run only by its queue outcome, which lives only under `.lym`, and nothing routes a run whose work the parent already holds to today's already-landed path; so a lost queue directory, or D6's backstop, which writes ledger landings but sets no outcome, leaves a landed member with no outcome, and it leads a batch whose `commit-tree` writes a second, empty landing commit. A landing then depends on state only `.lym` holds, against PATTERN-lym-files."
    fix: "Add to D3 and D8: before it acts on its outcome, a run that takes the lock reads `ParentHoldsTaskWork` for its task branch tip. A run whose work the parent holds takes today's already-landed path whatever its outcome says. Add to D4: an entry whose task head the chain commit before it already holds (`Fabric.HoldsWork`) is left out of the chain. Add to D6's backstop that it sets `landed`, with the landing commit, on each entry it writes a ledger landing for. Add to Constraints, under PATTERN-lym-files, that the queue is a cache: an absent entry or outcome reads as none, and a landed run is still found landed through the parent. Add an acceptance criterion: a run whose work the parent holds and whose entry is missing or has no outcome writes no landing commit and ends Finalize through the already-landed path. Tier 1, `internal/landingshed`."
  - id: F4
    severity: MEDIUM
    class: design
    origin: lens:walkthrough
    location: _lym/discussion/decision-record.md:82
    summary: "D6's backstop cannot always find the missing landings: it identifies a chain commit by an entry that is `landed` or a trailer naming a queued entry, but D2 lets the next leader prune a dead entry with no outcome first, and with that entry goes its join-time records commit; `lym fabric pull` and `lym fabric anchor` also take the parent lock without running any backstop, so a leader dying between the move and the ledger write can leave the parent ahead by commits that are no owed tip, and `syncParent` then stops every later Finalize."
    fix: "Make the leader write, before the move and under the lock, one durable mark of the move it is about to make: the chain base, each prefix commit and each member's slug and records commit. Write it in the queue directory next to the entries, and remove it only after every ledger landing and outcome is written. In D6, have the backstop read that mark rather than the entries, and run it before any pruning of dead entries (D2). State in D6 that a lock taker which is not a run, such as `lym fabric pull`, leaves a standing mark alone and the next Finalize writes the landings. Add an acceptance criterion: after a leader dies between the move and the ledger write, and its dead entries would be pruned, the next run to take the lock writes one ledger landing per moved commit with that member's records commit. Tier 1, `internal/landingshed`."
  - id: F5
    severity: MEDIUM
    class: design
    origin: lens:interactions
    location: _lym/discussion/decision-record.md:94
    summary: "When the culprit is the leader's own entry, D7 sets no outcome on it, and the leader's fixer runs with the lock freed; D3 then lets the next lock holder take that live entry with its unfixed head. A batch that passes marks it `landed`, and D8 ends the culprit's Finalize, so the fixer's commits never land."
    fix: "In D7, give a culprit that is the leader's own entry outcome `failed`, like any other culprit, so D8's own-path bullet covers it and no leader takes it. Add an acceptance criterion: while a leader that was its batch's culprit runs its fixer with the lock freed, the next leader's batch does not take its entry. Tier 1, `internal/landingshed`."
  - id: F6
    severity: MEDIUM
    class: design
    origin: lens:walkthrough
    location: _lym/discussion/decision-record.md:105
    summary: "D8 makes a `conflict` run merge the parent under the lock through the resolver. The conflict session therefore runs while the run holds the parent lock: `freeLock` frees it only for a fixer or a fix review, and `finalizeKeepsLock` keeps it past the idle bound, up to `landing.conflict_timeout_min`. D1's rationale says a conflict session must never hold the lock."
    fix: "Change D8's `failed` or `conflict` bullet so that a `conflict` run frees the lock and merges the parent's tip into its task worktree with no lock held, through the resolver, as D1's merge does. It keeps its outcome, then takes the lock again at the front of the queue and runs today's merge, gate and fixer alone. Add an acceptance criterion: a `conflict` run's conflict session runs with no parent lock held. Tier 1, `internal/landingshed`."
  - id: F7
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md:52
    summary: "D3 checks a member's pause, liveness and task tip only when the batch is taken, while the lock is then held across the batch gate and any bisection gates before the move; a pause that comes to cover a member, or a commit on its task branch, in that time still has its old head moved onto the parent and marked `landed`, and its newer commits never land."
    fix: "Add to D6: right before the guarded move, the leader re-reads each member's pause cover, liveness and task branch tip. A member that fails any of them is left out, and the leader rebuilds and gates the chain without it, as for a moved parent head. The rebuild is bounded as F16 asks. Add an acceptance criterion: a member paused, or whose task branch moved, after the batch was taken is not moved onto the parent. Tier 1, `internal/landingshed`."
  - id: F8
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md:43
    summary: "D2 says a run removes its own entry when its Finalize ends, but not whether a halt or Stuck is an end, nor whether a run rejoining at Finalize rewrites or keeps its entry; D8's `landed` test compares only the landing commit, so a kept `landed` outcome over a task branch with new commits, after a `lym loom goto` back, ends Finalize without landing them, and a dropped `failed` outcome lets an unjudged head be batched."
    fix: "In D2, say which endings remove the entry: Done removes it, and a halt or Stuck keeps it. Say that a run joining again at Finalize rewrites the entry with its new head and keeps the outcome only while the task branch tip is still the entry's task head. In D8, make the `landed` bullet need both the landing commit on the parent and the task branch tip equal to the entry's task head; otherwise the run clears the outcome and acts as with none. Add an acceptance criterion: a run whose task branch gained commits after it was marked `landed` lands them. Tier 1, `internal/landingshed`."
  - id: F9
    severity: MEDIUM
    class: design
    origin: lens:existing-paths
    location: _lym/discussion/decision-record.md:68
    summary: "D5 binds only reads on the batch path, but other readers of the leader's worktree HEAD run while it is detached: the pair's pusher (`readLocalBranch`, the push's `CurrentBranch`) and `RefusedPushStands` error or read the code side as unpushed, and `ParentHoldsTaskWork` and the darn tree scope read HEAD. The rule's reach is undefined: applied to the darn scope read, a batch gate tests only the leader's own trees."
    fix: "In D5, name the readers and what each does while the worktree is detached. The pair's pusher and its refusal read hold off while the leader holds the pair push lock, which the leader takes across each detach and restore. `ParentHoldsTaskWork` and the owner checks resolve the task branch ref. The gate's scope read (`darnTreeScope`) keeps reading HEAD, so a tree-scoped leader's batch gate scopes to the trees every member of the prefix touches. Add that last point to D5's Bound. Add an acceptance criterion: a darn leader's batch gate over a chain commit runs targets for every member's trees. Tier 1, `internal/landingshed`."
  - id: F10
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md:69
    summary: "D5 restores a detached task worktree only at Finalize's entry and resume, but a leader that died while detached can be sent with `lym loom goto` to an earlier row. That row then runs in a detached worktree, where fabric's merge guards refuse and an agent session's commits are reachable from no ref."
    fix: "Move D5's restore from Finalize's entry and resume into the run's resume itself, the step every row's resume and `lym loom goto` pass through, so a clean worktree detached by the batch is put back on its task branch before any row runs. Mark a batch detach so the restore acts only on a detach the batch made: the leader writes a scratch marker naming the chain commit before it detaches and removes it after the restore. Without the marker, a detached worktree is Stuck naming the commit. Change D5's criterion to a resumed run at any row."
  - id: F11
    severity: MEDIUM
    class: design
    origin: lens:walkthrough
    location: _lym/discussion/decision-record.md:67
    summary: "D5's checkout of a chain commit can leave the leader's ignored build output untracked, where a member deleted a directory or its ignore rule, or the reverse on the restore. `verifytree.Verify` then returns dirty and gates nothing, and the restore meets a dirty tree. Today's merge-in removes such leftovers before each clean check, and the batch checkouts have no counterpart."
    fix: "Add to D5 that the leader removes, after each checkout and after the restore, the ignored files left in directories the checkout deleted since the commit it left, through the same seam the merge-in uses (`Deps.RemoveLeftovers`), before the clean check. Add an acceptance criterion: a batch whose member deletes a directory holding ignored build output gates the chain commit and restores the leader clean. Tier 1, `internal/landingshed`."
  - id: F12
    severity: MEDIUM
    class: consistency
    origin: lens:existing-paths
    location: _lym/discussion/decision-record.md:83
    summary: "D6's Bound says a member's records commits from its join on are held back until its landing is pushed, but the only hold is a push-ledger landing, which the leader writes at the move; between join and move the member's pusher may push the join-time records commit and later ones, and the ledger then holds a commit origin already has."
    fix: "Rewrite D6's Bound to say the hold starts at the leader's ledger write. At the move, the leader records each member's oldest records commit that origin lacks, falling back to the member's records head, as `oldestHeldRecords` picks it today. Change D2 so the entry's join-time records commit is only that fallback. No join-time records commit marks the run landed, so an earlier push is harmless; state that as the Bound's reason."
  - id: F13
    severity: MEDIUM
    class: consistency
    origin: lens:interactions
    location: _lym/discussion/decision-record.md:170
    summary: "`## Acceptance criteria` names no test for D5's dirty-detached worktree being Stuck with its paths, D7's culprit-is-the-leader path (it lands nothing and runs today's merge, gate and fixer alone, the other entries kept with no outcome), D2's removal of a run's own entry at its Finalize's end, or D6's held-back records push."
    fix: "Add these acceptance criteria, each Tier 1, `internal/landingshed`: D5 — a Finalize resumed with its task worktree dirty and detached is Stuck naming the dirty paths, and moves nothing; D7 — a batch whose culprit is the leader lands nothing, gives the leader outcome `failed` (F5) and leaves the other entries with no outcome; D2 — a run that ends Finalize at Done leaves no queue entry; D6 — a member's pair pusher sends nothing past the records commit the leader's ledger landing holds until the parent's push is made."
  - id: F14
    severity: LOW
    class: design
    origin: lens:walkthrough
    location: _lym/discussion/decision-record.md:61
    summary: "D4 takes each landing message from the entry, fixed at join, so a batch-gate failure that passes on its rerun shows in no landing commit, where today every landing commit lists it under its flaky section. The record also leaves open how the batch gate runs the rerun without writing the leader's landing failure record or its in-process flaky list."
    fix: "Add to D4 that batch-gate failures a rerun did not repeat are listed nowhere in the landing commits, since the commits are built before the gate, and are logged at Warn with the chain commit instead. Add to D7 that the batch gate runs `verifyGate`'s check and rerun with no failure recorder and no flaky callback, so it writes no landing failure record and adds nothing to the leader's flaky list."
  - id: F15
    severity: LOW
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md:89
    summary: "D7 rests its blame on premises the code does not keep: the empty prefix counts as passing, though `syncParent` can fast-forward the parent onto upstream commits no gate here ran on; and `after verifytree's one rerun` names landingshed's rerun, which reruns test-run failures only, so a tier timeout or a build failure starts a bisection with no rerun."
    fix: "In D7, name the rerun as landingshed's rerun of a test-run failure, and add to its Bound that a failure from ungated upstream commits on the parent, or a tier timeout or build failure the rerun does not repeat, blames entry 1 or the prefix where it first shows. The blamed run's own gate then runs on its own path, as for any culprit, so nothing lands ungated."
  - id: F16
    severity: LOW
    class: consistency
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md:115
    summary: "The record's gate counts disagree with its mechanism and with the support log: D9 says a batch of four costs at most two extra gates, support-log Q7 says three, D7's culprit-is-the-leader path gates again a tree its prefix-1 gate already found failing, and D6's rebuild after a moved parent head has no bound."
    fix: "In D7, send a leader that is its own batch's culprit straight to its fixer on the prefix-1 gate's log, writing the landing failure record from that log, so no gate runs twice on one tree. In D6, bound the rebuilds after a moved parent head, a left-out member (F1, F2, F7) included, at `landing.batch_max` rebuilds per lock hold, after which the leader lands alone. Restate D9's rationale per chain built. In `_lym/discussion/support-log.md` Q7, change three to two."
  - id: F17
    severity: LOW
    class: consistency
    origin: lens:walkthrough
    location: _lym/discussion/decision-record.md:104
    summary: "D8's second bullet blames a `landed` outcome whose landing commit is not on the parent on a leader that died before the move, but D6 writes `landed` only after the move; only a parent moved back by something outside the queue reaches that bullet, and its acceptance criterion names the wrong case too."
    fix: "Reword D8's second bullet to name the case that reaches it, a parent whose head no longer holds the landing commit, such as after an operator's reset. Reword the D6, D8 criterion's last clause the same way."
  - id: F18
    severity: LOW
    class: consistency
    origin: lens:existing-paths
    location: _lym/discussion/decision-record.md:114
    summary: "D9 and its criterion say a hub layer still setting `landing.finalize_lost_races` is warned of as a retired key, but in a default or worktree layer configengine warns of such a key as an unknown key naming `lym config reconcile`, and `RetiredKeys` changes only the local and machine layers' refusal."
    fix: "In D9, say the key joins landing's `RetiredKeys` as CategoryCrossPair, so the local and machine layers refuse it, and a default or worktree layer still setting it gets configengine's unknown-key warning. Change the D9 criterion's last clause to: a hub default layer setting `landing.finalize_lost_races` loads with the unknown-key warning."
---

### [BLOCKING] Batched members skip their own under-lock checks

**Location:** `_lym/discussion/decision-record.md:77` (D6), with D3's skip rule at :52.

**Issue:** `landUnderLock` runs `ownerChecks`, `parentHoldsWork`, `landCheck` and, at the record, `handedOverStop` for the run that lands (`internal/landingshed/finalize.go:473-491`, :1035; `finalize_owner.go:40-70`).
In a batch only the leader runs them.
PATTERN-owner-machine lets a landing proceed only when the run's entry does not read as owned by another machine, and the reconcile also stops an ended or stopped entry.
A member adopted elsewhere, ended on origin, or a beam with open members can therefore be moved onto the parent.
The parent's pusher would catch an owner change later, but by then the local parent already holds the landing.

### [BLOCKING] The pull-request decision for a member is made outside the lock

**Location:** `_lym/discussion/decision-record.md:32` (D1), with D10 at :119.

**Issue:** The declared PATTERN-landing text binds "deciding on a pull request: only in a run's Finalize under `internal/parentlock`'s lock".
D1 has each run resolve whether it lands through a pull request before it writes its entry and waits.
A member the leader lands never takes the lock to decide again.
Today the decision is re-read under the lock: `pullRequestLanding` and the beam record check there exist because "a landing another run recorded while this one waited for the lock is seen here" (`finalize.go:456-467`).

### [BLOCKING] A landing depends on a queue outcome held only under `.lym`

**Location:** `_lym/discussion/decision-record.md:82` (D6's backstop), with D2 at :44 and D8 at :103-106.

**Issue:** D2's rationale relies on an outcome outliving a driver restart, and D8 routes only by outcome.
PATTERN-lym-files allows nothing a landing depends on to live only under `.lym`, where an absent cache reads as empty.
Today `ParentHoldsTaskWork` sends a run whose work the parent holds to the already-landed path (`finalize.go:371-377`, :473-486), and `Fabric.Merge` reads an up-to-date squash as committing nothing (`internal/fabricengine/merge.go:444-450`).
The record keeps neither guard on the batch path.
`commit-tree` always writes a commit.
So a member whose outcome is missing, after a lost queue directory or after D6's backstop, which sets none, leads a batch over its own landed work and lands an empty second commit.

### [MEDIUM] The backstop loses landings when dead entries are pruned first or a non-run takes the lock

**Location:** `_lym/discussion/decision-record.md:82`.

**Issue:** The backstop finds a moved commit through its entry.
D2 lets a leader remove a dead entry with no outcome, and the record orders neither step.
Once the entry goes, so does its join-time records commit, and no ledger landing can be rebuilt.
`lym fabric pull` and `lym fabric anchor` take the same lock and run no backstop.
`syncParent` then refuses a parent ahead by commits that are no owed tip (`finalize.go:739-749`).

### [MEDIUM] A leader that is its own culprit can be batched by the next leader

**Location:** `_lym/discussion/decision-record.md:94`.

**Issue:** D7 marks only another run's culprit `failed`.
The leader's fixer runs after `freeLock` (`finalize.go:1216`).
D3 lets the next lock holder take the leader's live entry with no outcome while its task tip is unchanged.
If that batch passes, D8 ends the culprit's Finalize as `landed`, and the fixer's commits are left unlanded.

### [MEDIUM] A `conflict` run holds the lock through its conflict session

**Location:** `_lym/discussion/decision-record.md:105`.

**Issue:** The run's own path reaches `resolver.Resolve` inside `mergeParent` (`finalize.go:1404`).
The lock is freed only for a fixer or a fix review (:1216, :1303).
`finalizeKeepsLock` keeps the lock while Finalize runs (`internal/loomcli/looplock.go:202`).
D1's rationale says no conflict session holds the lock, yet the run sure to need one now holds it.

### [MEDIUM] Membership is not re-checked before the move

**Location:** `_lym/discussion/decision-record.md:52`.

**Issue:** The skip rule runs once, at the take.
A batch can then hold the lock across three or more gates of 10 to 15 minutes each.
A pause, a dead member or a moved task tip in that window still lands the member's old head, and D8 then ends its Finalize.

### [MEDIUM] Entry lifecycle across halts and rejoins is unsettled

**Location:** `_lym/discussion/decision-record.md:43`.

**Issue:** "When its Finalize ends" does not name halts.
D1's join does not say whether it rewrites an entry or keeps it.
D8's `landed` test reads only the landing commit, while today's held check reads the task head (`internal/fabricengine/replay.go:338-350`, per the bound-breaker lens).
One choice loses a `failed` outcome, and the other ends Finalize over new commits.

### [MEDIUM] Other readers of the detached worktree's HEAD

**Location:** `_lym/discussion/decision-record.md:68`.

**Issue:** The existing-paths lens traced the pair pusher's `readLocalBranch` and the push's `CurrentBranch`, which error or misread on a detached HEAD.
`ParentHoldsTaskWork` and `darnTreeScope` (`internal/loomcli/darnwiring.go:105`) read HEAD as well.
D5 binds only "the batch path", so which of these follow the detach is left to the plan.
For the darn scope read the choice decides whether members' changes are tested at all.

### [MEDIUM] The restore misses runs sent to an earlier row

**Location:** `_lym/discussion/decision-record.md:69`.

**Issue:** Way forwards in this code send the operator through `lym loom goto --to <row>`.
A leader that died while detached and is sent back that way runs its row detached.
Fabric's merge guards refuse a detached HEAD (`internal/fabricengine/mergeguards.go:162`).
The restore also cannot tell a detach the batch made from one an operator made.

### [MEDIUM] Checkout leftovers dirty the batch gate

**Location:** `_lym/discussion/decision-record.md:67`.

**Issue:** `verifytree.Verify` returns dirty on any untracked path (`internal/verifytree/verify.go:359-365`).
Ignored output in a directory a member deleted becomes untracked after the checkout.
Finalize's merge-in clears such leftovers (`finalize.go:1467-1493`), and D5 gives its checkouts no counterpart.

### [MEDIUM] The records hold "from its join on" has no mechanism

**Location:** `_lym/discussion/decision-record.md:83`.

**Issue:** `recordsPushCeiling` reads a hold from a ledger landing alone (`internal/fabricengine/pushanchored.go:98-110`; `internal/pushledger/ledger.go:239-261`).
The leader writes that landing at the move, so nothing holds a member's records between join and move.

### [MEDIUM] Acceptance criteria leave added behaviours untested

**Location:** `_lym/discussion/decision-record.md:170`.

**Issue:** The criteria name no test for D5's dirty-detached stop, D7's culprit-is-the-leader path, D2's entry removal or D6's held-back push.
Each of these is a behaviour a Decision adds, and the discussion spec requires each new test's intent and tier.

### [LOW] Batch-gate flaky failures and records are unsettled

**Location:** `_lym/discussion/decision-record.md:61`.

**Issue:** `landingTexts` appends the flaky section of this process's gate failures to the landing message (`finalize.go:172-202`).
D4's messages are fixed before the batch gate, and the record says neither how the batch gate keeps out of the leader's failure record and flaky list nor where its flaky failures go.

### [LOW] D7's blame premises

**Location:** `_lym/discussion/decision-record.md:89`.

**Issue:** `syncParent` fast-forwards the parent to its upstream (`finalize.go:752-772`), so the parent can hold upstream commits no gate here ran on.
The rerun is landingshed's, and it runs only for test-run failures (`internal/landingshed/verifygate.go:306-318`).
A timed-out tier is a plain failure (`verify.go:461-470`).
Today a lone run meets the same causes; the batch moves them onto entry 1 or the first failing prefix, which the Bound does not say.

### [LOW] Gate-count claims

**Location:** `_lym/discussion/decision-record.md:115`, and `_lym/discussion/support-log.md:74`.

**Issue:** D9 says two extra gates for four entries, and Q7 says three.
The leader-culprit path re-gates the failing prefix-1 tree, since verifytree records passes only (`verify.go:487`).
D6's rebuild after a moved head has no bound.

### [LOW] D8's second bullet names an impossible cause

**Location:** `_lym/discussion/decision-record.md:104`.

**Issue:** D6 sets `landed` only after the move, so a leader that died before the move leaves no `landed` outcome.

### [LOW] The retired-key warning does not exist in the hub layer

**Location:** `_lym/discussion/decision-record.md:114`.

**Issue:** In the hub's default layer, configengine warns of a retired key as an unknown key (`internal/configengine/config.go:324-326`; `doc.go:71-75`).
