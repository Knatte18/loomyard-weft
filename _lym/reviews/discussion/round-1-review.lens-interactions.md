```yaml
lens: interactions
outcome: reported
```

Task: the interactions-lens review of `decision-record.md` and `support-log.md`. I checked each pair of decisions that act on the same state, then checked the criteria and constraints against them.

**Pairs checked**
- D1 × D3 (lock-free merge-in, then the leader syncs the parent and the chain is based on the new head): held.
- D1 × D8 `conflict` (D1's rationale keeps conflict sessions out of the lock; D8 runs the resolver under the lock): broken (F2).
- D1 × D10 (pull-request need resolved before joining, versus today's re-read under the lock): broken (F6).
- D2 × D6 backstop × D8 (outcome writes, the backstop's ledger writes, dispatch on outcome): broken (F3).
- D2 entry removal × D7/D8 `failed` kept until the run lands alone: broken, low (F8).
- D2 join-time records commit × D6 hold "from its join on": broken (F4).
- D3 batch take × D7 culprit is the leader (no outcome set on its own entry): broken (F1).
- D3 batch take × D6 owner checks for members × `PATTERN-owner-machine`: broken (F5).
- D4 messages fixed at join × D7 batch-gate rerun and flaky passes (`Finalize.flaky`): broken, low (F7).
- D5 detach and restore × D7 several bisection checkouts: held.
- D5 restore × D7 culprit-leader's own merge: held, since D5 restores "on every way out".
- D5 resume restore × D8 resumed run: held.
- D6 "head moved → build and gate again" × D7 gate-count claims × D9 rationale: broken, low (F9).
- D6 landed size from first parent × D8 member doing its remaining records: held, through the entry's landing commit.
- D6 × D9 `batch_max` = 1: held (D3's batch of one runs today's path).
- D6 × D10 (a pull-request run lands alone under the same lock): held.
- D7 "no landing failure record" × D5 gate through `verifyGate.check`: held, provided the plan bypasses `recordFailure`. The record doesn't name the mechanism (folded into F7).
- D9 × D3 cap: held. D9 rationale "two extra gates" × support-log Q7 "three extra gates": broken, low (F10).
- D12 PATTERN text × D6 (the leader lands other runs inside its own Finalize): held.
- Constraint on idle release × D8 `conflict` (`finalizeKeepsLock` keeps the lock through a conflict session): broken (part of F2).
- Constraint `PATTERN-daemon-wakeups` × D8 (a landed member takes the lock only to read its outcome): held.
- Acceptance criteria × D5, D6 and D7 added behaviours: broken (F11).

**Findings**

F1. BLOCKING, design. Location: decision-record.md D7, against D3.
- **Summary:** When the culprit is the leader's own entry, D7 sets no outcome on it, and the leader then runs its fixer with the lock freed. D3 lets the next lock holder take any live entry with no outcome and an unmoved task tip. So the next leader can batch the culprit's unfixed head while its fixer is running. If that batch passes (the D7 bound allows an interaction-only failure), the culprit is marked `landed` at its pre-fixer head. When the culprit comes back, D8 `landed` ends its Finalize, so the fixer's commits never land.
- **Evidence:** decision-record.md:94 (no outcome for the leader-culprit), :51-52 (D3 take rule), :103 (D8 `landed`); finalize.go:1216 (`runFixer` calls `freeLock` first).
- **Fix:** Give the leader's own culprit entry outcome `failed`, so D8's own-path rule covers it. Add a criterion for it.

F2. MEDIUM, design. Location: D8 `conflict` bullet, against D1 and the idle-release constraint.
- **Summary:** D1's rationale keeps conflict sessions out of the lock. D8 makes a `conflict` run, whose own lock-free merge-in was clean, run the resolver and its conflict session under the lock. `finalizeKeepsLock` keeps the lock for the whole session (`landing.conflict_timeout_min` is 60), blocking every landing on the parent.
- **Evidence:** decision-record.md:36, :105, :149; looplock.go:202; finalize.go:1404 (the resolver runs inside `mergeParent`, and the lock is freed only for the fixer and the fix review, finalize.go:1216, :1303).
- **Fix:** Have a `conflict` run free the lock, merge the new parent tip lock-free through the resolver, then take the lock again at the front and land alone.

F3. MEDIUM, design. Location: D6 backstop, against D8 and D3.
- **Summary:** The backstop writes the missing ledger landings but sets no outcome. A member whose leader died after the move, before any outcome write, wakes with no outcome. D8 then sends it to lead a batch, so D4 chains its already-landed head. `commit-tree` writes a new empty landing commit, unlike `Fabric.Merge`'s up-to-date classification. The record never sends a run with no outcome, whose work the parent already holds, to today's already-landed path.
- **Evidence:** decision-record.md:79, :82, :106, :60-62; finalize.go:283 (the empty squash is classed as already up to date), :473-486; parenttips.go:225.
- **Fix:** Have the backstop also set outcome `landed`, with the landing commit, on each entry it recovers. State that an entry whose head the chain base already holds (`HoldsWork`) is left out of the chain and takes the already-landed path.

F4. MEDIUM, consistency. Location: D6 Bound, against D2.
- **Summary:** D6 says a member's records commits "from its join on are held back". The only hold that exists is the push ledger's landing, which the leader writes only at the move (`recordsPushCeiling` reads `HeldRecordsCommit`). Between join and move, the member's pusher can push the join-time commit and later ones, so the bound states a hold that no decision creates.
- **Evidence:** decision-record.md:79, :83; pushanchored.go:98-110; ledger.go:239-261; landingdeps.go:982-1011 (`oldestHeldRecords` drops a hold that origin already has).
- **Fix:** Either restate the bound as held from the ledger write on, which is enough since no join-time records commit marks the run landed, or add a decision that holds the commit from join.

F5. BLOCKING, design. Location: D6 "today's owner checks", against D3's skip list and constraint `PATTERN-owner-machine`.
- **Summary:** The constraint says every landing proceeds only when the run's entry does not read as owned elsewhere, ended or stopped. Today each run checks this under the lock through `ownerChecks` (run-scope reconcile and hand-over) and `landCheck`. In a batch only the leader's checks run, and D3 skips only paused runs and moved task tips. A member adopted by another machine, ended or stopped on origin, or a beam with open members, is landed by the leader.
- **Evidence:** decision-record.md:52, :77, :144; finalize_owner.go:40-70; finalize.go:473-491 (owner and land checks per run under the lock); pattern/PATTERN-owner-machine.md (rule line).
- **Fix:** Have the leader run each member's run-scope owner and land checks under the lock before the move, and leave out any member that fails them.

F6. MEDIUM, design. Location: D10, against D1.
- **Summary:** D1 settles whether a landing needs a pull request before joining, with no lock held. Today Finalize re-reads under the lock (`pullRequestLanding`) because "a landing another run recorded while this one waited for the lock is seen here". A pull-request need that appears after a member joined, such as a beam's pull-request record or a `trail` mode change, still has the member batched and landed locally. D10's bound covers only GitHub rules the resolver's read misses.
- **Evidence:** decision-record.md:32, :119-121; finalize.go:458-467.
- **Fix:** Have the leader re-read each member's pull-request need under the lock, using the resolver's on-disk read, and leave out any member that now needs GitHub.

F7. LOW, consistency. Location: D4 landing message, against D7 and D5.
- **Summary:** D4 takes each landing message from the entry, written at join. The batch gate's `onFlakyPass` rerun appends to the leader's in-process `flaky` list, so flaky failures of the batch gate show in no landing message. Today every gate failure a rerun did not repeat is listed in the landing commit. The record also says neither how the batch gate avoids the leader's `recordFailure` and `clearFailure`, nor how it avoids the rerun.
- **Evidence:** decision-record.md:61, :95; finalize.go:166-180; verifygate.go:286-296.
- **Fix:** State whether batch-gate flaky passes are listed, and in which commit's message, and how the batch gate skips the leader's failure record.

F8. LOW, design. Location: D2 entry removal, against D7 and D8 `failed`.
- **Summary:** "A run removes its own entry when its Finalize ends" doesn't say whether a halt (spent fixer, Stuck) counts as an end. If it does, a `failed` run that halted and resumed rejoins with no outcome and can lead a batch on a head nobody judged. If it doesn't, the entry stays until resume.
- **Evidence:** decision-record.md:43-44, :105.
- **Fix:** Name which endings remove the entry.

F9. LOW, consistency. Location: D6 "builds the chain again … and gates again", against D7 and D9.
- **Summary:** A head that moves mid-bisection restarts the whole chain and its gates, with no bound on how often. D9's "two extra gates" and D7's "about log2(n) extra gates … in the same lock hold" do not account for it.
- **Evidence:** decision-record.md:78, :98, :115.
- **Fix:** Bound the rebuilds, or state the claim per chain built.

F10. LOW, consistency. Location: support-log.md Q7, against D9.
- **Summary:** Q7 says the cap "bounds bisection at three extra gates", while D9 says two. With n = 4, the search over prefixes 1 to 3 takes at most 2 gates.
- **Evidence:** support-log.md:74; decision-record.md:115.
- **Fix:** Correct Q7 to two.

F11. MEDIUM, consistency. Location: `## Acceptance criteria`, against D5, D6 and D7.
- **Summary:** No criterion names a test for three behaviours the Decisions add:
  - D5's dirty-detached worktree being Stuck with its paths named;
  - D6's held-back records push of a member;
  - D7's culprit-is-the-leader path (lands nothing, then today's merge, gate and fixer alone, with the other entries left with no outcome).
- **Evidence:** decision-record.md:69, :83, :94, :167-170.
- **Fix:** Add a criterion, with intent and tier, for each.

**Tool calls:** none; the inherited context covered every check.
