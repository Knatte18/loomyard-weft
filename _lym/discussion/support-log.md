## Interview

### Turn 1 — exploration (autonomous mode, no operator)

The board entry carries the operator's decisions of 2026-11-26, which settle the pusher's scope, the adaptive interval, the summary notice, the reed field, the landing rule and the refusal rule.
No `Decision for push-retry-not-halt:` note is on the board.
Three subagents read the pusher, the detached pushes, the landing's pushes and GitHub reads, the driver's re-step rule, Seed-IDE, the reed bar, the agents' push instructions and Worktree-Teardown.
Facts that changed the design:

- The Seed-IDE row has no origin check; the refusal issue #218 quotes is the code-tip wait in `internal/runfsm/act.go`, which the pusher already lifts for a pair standing on this machine.
- The loomcli "status push is owed" loop issue #254 quotes no longer exists.
- The Go driver re-steps a network stop three times within about two minutes, then hands back.
- The pusher's owed retry runs from one second to a one-minute ceiling, with no config key, and posts one notice per call at the ceiling.
- The board, `lym fabric sync` and `lym fabric add` push through their own detached spawns, outside the pusher.
- Finalize pushes the parent synchronously under the parent lock.
- Worktree-Teardown halts after three push rounds that leave a tip off origin.

### Turn 2 — design picks

Every open question was settled by auto-pick; see the question ledger.

### Turn 3 — lens review

All six lenses returned a report.
Duplicates across lenses are merged; each line names the lenses that raised it.

- D7 presence test defeated by fetches (bound-breaker, walkthrough, existing-paths; BLOCKING): fixed, D7 now tests reachability from a local branch or the code branch's reflog, excluding remote-tracking refs; red-first criterion rewritten.
- D7 takeover makes a reset refuse (interactions F7): fixed, D7 states the stop as accepted.
- D7 "every rejection is a refusal" overstates the code (bound-breaker): fixed, the takeover read and rebase-free retry come first.
- D6 dobby run watch mails per re-step, `4aa85ab3a` (bound-breaker, existing-paths 9; BLOCKING): fixed, a wait record under a held run lock is read as waiting by the run watch, `halt.go` and `lym loom status`.
- D6 dead driver looks like a live wait (walkthrough 6): fixed, a wait record with a free run lock is reported once as a stop.
- D6 vs D3 notice count, 1 + 2N mails (interactions F2): fixed, the driver posts no notice; waiting runs join D3's one summary.
- D6 `ErrOwnerOriginUnreachable` class reaches catch-up and adopt (bound-breaker, walkthrough 8): fixed, Finalize's owner check wraps it and the error is unchanged.
- D5 Finalize's synchronous records push (interactions F1; BLOCKING): fixed, the records commit goes through the pair's pusher.
- D5 board `done` may land before the parent push (existing-paths 2; BLOCKING): fixed, the landing target pushes the parent first and `done` after.
- D5 past-the-merge read reads origin's parent (existing-paths 3): fixed, it reads the local parent while a landing push is owed.
- D5 pull-request wait has no mechanism or bound (interactions F5, walkthrough 5): fixed, a `git-transport` stop before the parent lock, and a refused parent push is a blocked halt.
- D5 local landing after a pull-request landing on a stale parent (bound-breaker): fixed, Finalize fast-forwards a lagging local parent first; a diverged one is a refusal.
- D5 and teardown "push at once" loses to a running pusher (interactions F4, walkthrough 4): fixed, D2 adds a push-now mark the running pusher's wait wakes on.
- D5 teardown split and landed-pair guard stated as existing and missing the parent tip (bound-breaker, existing-paths 4, leanness): fixed, new D13 with its own criterion and the parent-owed check.
- D2 interval not kept across pusher processes (bound-breaker, walkthrough 3, existing-paths 10): fixed, the ledger keeps the last attempt and interval per target.
- D3/D4 "owed" meant only a failed push, missing plain commits (interactions F3, bound-breaker, walkthrough 2-3, existing-paths 7): fixed, owed means ahead of origin, written by the pusher and the dobby's ahead check; D1 states the ahead rule for plain and beam merge commits.
- D3 hub-wide clear time races (interactions F3, bound-breaker): fixed, one ledger lock guards the first add, the last remove and the hub file.
- D3 stale entries for removed targets (existing-paths 6): fixed, removal, hand-over, cancel and a dobby tick drop them.
- D3 refused target stays in the ledger (interactions F6): fixed, a refused target leaves it.
- D3 no signal when the prime's dobby is down (bound-breaker, LOW): rejected, the prime's dobby is the hub's one watcher and its absence stops far more than notices; kept as an auto-mode assumption.
- D1 webster reset loses its owed hold (existing-paths 5): fixed, webster reset's records push stays outside the pusher.
- D1 board pushes whose result a caller reads (existing-paths 8): fixed, the owner write, hubreconcile and tender keep their synchronous push.
- D9 pre-merge list lost on a resume (bound-breaker, walkthrough 7): fixed, the rule reads the merge commit's first parent, so it runs on every resume.
- D4 how the `#()` job finds its hub (walkthrough 9): fixed, a server-global option set at server start.
- D4/D3 hub instead of machine (board-scope 1, class decision): raised to the parent in Open risks; the hub pick stays meanwhile.
- D10 gate claim unverified (board-scope 2): fixed, a grep found no production file under the gate packages reading origin; D10 names them.
- D12 #254's bound not delivered (board-scope 3): fixed, D12 says the quiet retry replaces it.
- Catch-up fetch failure still stops a start (board-scope 4): added to Open risks.
- Constraints duplicate Goal and Decisions (leanness 1-3): fixed, Goal drops the mechanism details and Constraints drops the bullets the decisions state.
- D4 rationale repeats the assumption (leanness 4): fixed, cut.
- Open risk on `fabric-core-push-origin` sequencing (leanness 7): fixed, cut, since the board's dependency shows it.
- D12 and D6 length (leanness 5-6) and the interview narrative (leanness 8): rejected, the lens itself found no cut.

## Rejected alternatives

- A new shed state for a network stop the driver waits on: rejected because every reader of the shed states, the goto and activity rules among them, would need it; D6 keeps `failed` with its transient class and changes what the driver and `lym loom status` do with it.
- Retrying a network failure inside the step: rejected because Finalize's step holds the parent lock, so a long wait inside it would stall every other landing on that parent.
- Keeping the Go driver's three-wait cap and raising its waits: rejected because the operator decided that a landing needing GitHub never fails on a transient error.
- Making Worktree-Teardown wait for the push without bound: rejected because it holds the batten run on one row, where leaving the pair to the dobby's landed-pair teardown ends the run and still removes nothing unpushed.
- One ledger per machine across hubs: rejected because each hub has its own origin and orch, and a hub's bar reads only its own.
- Marking lym's commits with a trailer to tell them from hand commits: rejected because agents' code commits carry no trailer; reachability from a local branch or reflog is the test lym can apply to every commit.
- Presence in this machine's object store as the hand-commit test: rejected in the lens review, since every fetch copies origin's commits in.
- Removing every untracked path after the merge-in: rejected because a file git did not ignore may be work, so D9 removes only previously ignored files under a deleted directory.
- A list of ignored paths taken before the merge-in: rejected in the lens review, since a resume after the merge has none; D9 reads the rules of the merge record's pre-merge task head instead.
- Per-run driver notices beside the dobby's summary: rejected in the lens review, since N waiting runs would mail 2N times, the per-item mail the operator ruled out.

## Review rounds

### Round 1

The review raised 27 findings; every one is fixed in the decision record.

- F1: D5 now waits (D6) at its owner reads until origin answers, as PATTERN-owner-machine requires; the Goal and the D5 criterion say so.
- F2, F6: D5 lands on a parent ahead only by owed tips, and every pass that finds the work merged locally but not on origin records the landing entry.
- F3, F7: the parent target's pusher makes the board `done` write once the parent push landed, and the landing entry holds the run's records commit until then.
  The self-check showed that a hold on the board target leaks through the synchronous board pushes and freezes the board behind a refusal, so the `done` write moved.
- F4, F8, F9: a refused target keeps its entry marked refused; "owes a parent push" means the landing entry stands; a long wait record triggers the summary too.
- F5, F13: D6 names the driver lock, and the step writes no halt note for a network-class stop.
- F10, F14: D13 removes the pair as today with pushes off; a beam parent has no landing target of its own.
- F11: D1 sorts pushes by kind; hubreconcile's board push is a `boardengine.SpawnSync` spawn, so it moves to the pusher instead of staying synchronous.
- F12, F27: D2 names the three keys and replaces `PushUntilLanded`'s loop and the transport backoff with one push per interval.
- F15, F16, F25: D9 takes its base from the merge record, runs before the pre-merge clean check, evaluates the rules with `git check-ignore --no-index` in a scratch repository, and states what its bound removes.
- F17, F20, F21, F22: D7 and D6 pin their git and classifier facts, name the known gaps and give the refusal a way forward.
- F18, F19, F23, F24, F26: the PATTERN text, the #238 red-first criterion, D8's pusher choice, the reproduction and the dropped Open risk.

### Round 2

The review raised six findings; every one is fixed in the decision record.

- F1: D5 gives batten's Land row the owed-tip rule for its pull of the prime, and an unreachable origin there waits as `git-transport`.
- F2: D5 keeps one landing entry per landed run, settled in landing order; D3 keeps the parent target's ledger entry while any landing entry stands.
- F3: D5's records hold binds every push of the run's pair records, and D13 removes the pair only once the run's landing entry is gone.
  Batten's record in the prime's records is not held, since every run shares that branch; a reader of it at Land checks the parent instead.
- F4: D5 and PATTERN-owner-machine give the parent target's pusher an owner read before each parent push and `done` write.
  The landing target pulls the Bolt for each read instead of waiting on the run-scoped catch-up, which no landing target triggers; a parent owned elsewhere is a refusal with a record.
- F5: D3 reads the refused state from the refused-push record and drops any entry a dobby tick finds level.
- F6: D9 takes the base from the pre-merge task head read in the merge-in pass and writes the merge record before the after-the-merge-in clean check.

### Round 3

The review raised four findings; every one is fixed in the decision record.

- F1: D5 names each parent target's push in its refusal, `git push` in the prime's worktree for the prime's code branch and `lym fabric push` for a beam parent, and D3 and its criteria follow.
  A landing whose work origin's parent already holds is dropped by the pusher with no `done` write, its held records commit reaching no branch, and a run owned elsewhere is then handed over by the pusher.
- F2: D5 and PATTERN-owner-machine read the owner of every run a parent push carries, and that run's owner again before its `done` write; a run owned elsewhere holds back the whole parent push.
- F3: D13 makes every other teardown of a pair refuse while its run's landing entry stands, `lym fabric remove` with `--force` or `--remote` included.
- F4: D9 writes the pre-merge task head to a base file under the run's scratch directory before the merge-in, and a resume reads the base from it alone.

## Question ledger

- Q1 (auto-pick): which config module holds the push timing keys? Picked the fabric module, `fabric.push_interval_s` and `fabric.push_ceiling_s`, since fabric owns pushes and its template holds no timing yet.
- Q2 (auto-pick): who posts the summary notice? Picked the prime's dobby on its tick, since it already ticks hub-wide and mails the orch; a pusher sees only its own target.
- Q3 (auto-pick): is the bar's `sync:` field per hub or per machine? Picked per hub, since a hub has one origin.
- Q4 (auto-pick): the notice threshold's key and default: `fabric.push_notice_after_min`, 30, as the entry proposes, shared with the driver's network wait.
- Q5 (auto-pick): how does a pull-request landing meet a parent whose landing push is still owed? It waits for that push first, so GitHub never merges into a stale base.
- Q6 (auto-pick): what does Worktree-Teardown do with tips still off origin? It ends done with the pair standing, and the dobby's landed-pair teardown removes the pair once origin holds both tips.
- Q7 (auto-pick, revised in the lens review): how does `resetUnnamedTip` tell a hand commit from lym's? A commit no local branch or code-branch reflog reaches is not lym's here, so the push is refused.
- Q8 (auto-pick): does the catch-up's fetch failure also hand off? No: that fetch is the owner verdict PATTERN-owner-machine requires; only the owed push hands off.
- Q9 (auto-pick): F1 of `transient-stop-not-failed`, a retryable stop: the Go driver waits out a network stop with no halt note, and `lym loom status` reads the run as waiting, with no new shed state.
- Q10 (resolved by reading): issues #218 and #254 are settled by `async-run-pusher`; D12 names them in the change description.
- Q11 (auto-pick, round 1): should the `sync:` field and the all-clear cover every hub on the machine, as the entry's word "machine" says? Settled by Q3's pick: each hub has its own origin and orch, so "this machine" is read as this machine's hub.
- Q12 (open, for the parent): should a catch-up fetch failure on an unreachable origin still stop a run's start? Recorded under Open risks.
- Q13 (auto-pick): what owes a push? A target with a branch ahead of its origin tracking ref, so D3's ledger and D4's field count work no pusher has tried yet.
- Q14 (auto-pick): how does a landing or teardown push at once while a pusher sleeps? A push-now mark in the ledger entry that the pusher's wait watches.
