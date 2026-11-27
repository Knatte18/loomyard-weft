---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    origin: lens:existing-paths
    location: _lym/discussion/decision-record.md:124
    summary: "After a local landing on the prime's branch, batten's Land row pulls the prime (battencli/land.go landDeps Pull -> Fabric.Pull). With origin unreachable the fetch fails (pull.go:565-566). With origin reachable, a local main ahead of origin by the owed landing commit is no fast-forward and refuseUnsafeRewrite refuses it as a local commit no reflog reaches (pull.go:593-621). Either way Land ends Stuck 'pull failed' (battenshed/land.go:265-270), a blocked stop D6 never waits out. So every local landing whose push has not landed yet halts the run right after Finalize, against the Goal's 'only a refusal that retrying cannot fix stops anything'. The record never names the Land row."
    fix: "In D5, add the Land row's rule while the parent target owes a parent push: Land's pull of the prime reads a local parent ahead of origin by only tips the parent target records as owed as level and pulls no code side over it, and an unreachable origin at that pull is a git-transport stop D6 waits out instead of a Stuck. Name batten's Land row in Scope's 'In' list. Add a D5 criterion (internal/battencli or internal/battenshed, tier1): a batten run whose landing push is owed passes Land with origin reachable, and with origin unreachable stops as git-transport instead of Stuck."
  - id: F2
    severity: BLOCKING
    class: design
    origin: lens:interactions
    location: _lym/discussion/decision-record.md:113
    summary: "D5 defines one landing entry per parent target, 'the parent target's record of the owed parent tip', yet that entry also holds one run's records commit and carries that run's done write (:130-134). A second landing on a parent whose first push is owed leaves 'the parent target then owes the newer tip' (:122). If the newer tip replaces the entry, the first run's done write is lost and its pair hold is released before the parent push. If entries stack, the record never says so. D3's rule that an entry goes 'when the push lands' (:86) also conflicts with D5 keeping the landing entry until the done write, and the all-clear reads only the ledger."
    fix: "In D5, define one landing entry per landed run on the parent target. Each entry names the run, its landed tip, the records commit its pair target holds and its pending done write. A later landing adds its own entry, and the parent target pushes the newest owed tip. Once origin's parent holds an entry's tip, that entry's done write is made, its pair hold is released and the entry goes, in landing order. In D3, state that the parent target's ledger entry stays while any landing entry stands, so the all-clear waits for every done write. Extend the D5 second-landing criterion: both runs' done writes are made and both pair holds released once the newer tip lands, and neither before."
  - id: F3
    severity: BLOCKING
    class: design
    origin: lens:interactions
    location: _lym/discussion/decision-record.md:133
    summary: "D5's hold binds only the pair's pusher, so 'origin never reads the run's records or an ended entry whose parent lacks the run's work' (:135) is broken by every other path. The catch-up D1 and D8 keep synchronous pushes the pair whenever it is ahead (battencli/catchup.go:251-257, 265-275), and so do PR-Review's record commit (loomcli/landingdeps.go:473) and cancel's records push (battencli/cancelcmd.go:365). After such a push, origin's tracking refs hold both tips. D13's Worktree-Teardown then removes the pair 'as today', and the composite marks the landed entry done (PATTERN-pair-remote background, pair teardown), before or without the parent push, even one that stays refused. Batten's own Land, Worktree-Teardown and done transitions also reach origin through the prime's records (battencli/commitstatus.go:165-185), and the run reader reads a batten record at Land as PastParentMerge (pairteardown/runread.go:204-208, 270-271)."
    fix: "In D5, state that the landing entry's hold binds every push of the run's pair records, not only its pusher: the catch-up's PushRun (D8), the pull-request record commit and cancel's records push send nothing past the held records commit while the entry stands. In D13, state that Worktree-Teardown removes the pair only once origin's tracking refs hold both tips and the parent target owes no parent push for the run, and otherwise ends done and leaves the pair. For batten's records in the prime, either hold its transitions past Finalize behind the same entry, or narrow the :135 claim to the pair's records and state that a reader judging origin's batten record at Land or Worktree-Teardown also reads whether origin's parent holds the work. Add to the D5 ordering criterion: a catch-up during an owed landing push leaves Finalize's records commit and the board done off origin, and Worktree-Teardown leaves the pair standing."
  - id: F4
    severity: BLOCKING
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md:281
    summary: "PATTERN-owner-machine binds 'pushing or landing a run, a Finalize past its merge included: only on the entry's owner machine', and 'after absence, a catch-up from origin first'. D5 hands the rest of a Finalize past its merge, the parent push and the done write, to the parent target's pusher. The constraint at :281 keeps an owner verdict only for the pair pusher's run-scope reconcile. For the landing target, which pushes the prime's code branch and writes done, no owner read is stated, and nothing orders its push after the catch-up a wake requires. A machine that wakes after the operator adopted the prime elsewhere pushes main and writes done anyway."
    fix: "In D5 and the PATTERN-owner-machine constraint at :281, state the owner verdict the parent target's pusher reads before each parent push and before each done write. The landing target reads the prime's owner from a Bolt pulled from origin, and a beam parent reads the beam's owner through the beam pair's run-scope reconcile. It pushes only while no other machine owns the parent, and after an absence it runs only once the dobby's catch-up has run. A parent owned elsewhere stops the pusher with one notice naming `lym adopt`. Add a D5 criterion: with the prime owned by another machine, the landing target pushes nothing and makes no done write."
  - id: F5
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    carries: F4
    location: _lym/discussion/decision-record.md:88
    summary: "D3 now says that clearing the refused-push record 'removes the mark and lets the pusher start again', but names no writer that removes it. D7's way forward ends in `lym fabric push`, whose landing clears only the failed-push record (pushclassified.go landed -> j.clear()). The dobby's ahead check writes nothing for a marked target, and Start and Conditions refuse while RefusedPushStands (runpusher/start.go:122-125, run.go:65-68). So the mark outlives its record, the bar's minutes keep counting and the all-clear never posts. An owed entry likewise stays when a synchronous push outside the pusher (the catch-up, PR-Review, cancel, a hand push) makes its target level, since D3 drops an entry only when 'the push lands' or the worktree goes (:86)."
    fix: "In D3, read the refused mark from whether the target's refused-push record stands (fabricengine.RefusedPushStands) instead of storing it. State that the dobby tick drops the entry of any target its ahead check finds level, whoever pushed it. Add to the D3 refused criterion: after a hand `lym fabric push` clears the record, the next dobby tick drops the mark and the entry, and the all-clear posts."
  - id: F6
    severity: MEDIUM
    class: design
    origin: lens:walkthrough
    carries: F16
    location: _lym/discussion/decision-record.md:210
    summary: "D9 takes its base from 'the pre-merge task head the merge record keeps', but mergeParent writes the merge record only after the after-the-merge-in clean check (finalize.go:1360 vs 1388), and parent-align also writes one (parentalign.go:454). In the pass that merges in, the record on disk is the previous merge's or none. On a task's first merge-in with no earlier record, the removal finds no base and the issue #242 halt comes back. A pass killed between the merge-in and the record write leaves a resume the same stale base."
    fix: "In D9, state that the merge-in pass takes the base from the pre-merge task head it read before merging, the value the merge record keeps as task_head, and that mergeParent writes the merge record before the after-the-merge-in clean check, so a resume reads this merge's record. Add to the D9 red-first criterion a first merge-in with no earlier merge record."
prior:
  - id: F1
    status: resolved
    answer: "The Goal, D5 (:116-119) and the D5 and D6 criteria now agree that a local landing waits as git-transport until origin answers its owner reads; the Land row after it is new finding F1."
  - id: F2
    status: resolved
    answer: "D5 :122 lands on a parent ahead by only owed tips, and :319 adds the criterion; how two runs share the parent target is new finding F2."
  - id: F3
    status: resolved
    answer: "The parent target's pusher makes the done write once the parent push landed (:130-132), and D1 :60 keeps the board target unchanged."
  - id: F4
    status: open
  - id: F5
    status: resolved
  - id: F6
    status: resolved
  - id: F7
    status: resolved
    answer: "The pair pusher's hold is stated (:133-134); its bypass by synchronous pushes is new finding F3."
  - id: F8
    status: resolved
  - id: F9
    status: resolved
  - id: F10
    status: resolved
  - id: F11
    status: resolved
    answer: "The fixed-differently reason holds: hubreconcile's board push is a SpawnSync spawn (walk.go:218), so the by-kind rule moves it."
  - id: F12
    status: resolved
  - id: F13
    status: resolved
  - id: F14
    status: resolved
  - id: F15
    status: resolved
  - id: F16
    status: open
  - id: F17
    status: resolved
  - id: F18
    status: resolved
  - id: F19
    status: resolved
  - id: F20
    status: resolved
  - id: F21
    status: resolved
  - id: F22
    status: resolved
  - id: F23
    status: resolved
  - id: F24
    status: resolved
  - id: F25
    status: resolved
  - id: F26
    status: resolved
  - id: F27
    status: resolved
---

### [BLOCKING] batten's Land row stops every local landing whose push is still owed

**Location:** `_lym/discussion/decision-record.md:124` (D5)
**Issue:** Raised by existing-paths as F-EP1, and by interactions (F-I4), bound-breaker (F-b), walkthrough (W2) and the handler.
I confirmed `pullWarpArm` in `internal/fabricengine/pull.go:563-614`. It fetches first, a local head that isn't a fast-forward of upstream goes to `refuseUnsafeRewrite`, and that refuses a local commit no reflog value reaches.
Land runs right after Finalize returns done, while the pusher has only just been woken.
So the stop hits in the ordinary case, not only during an outage.

### [BLOCKING] One landing entry per parent target cannot carry two runs' holds and done writes

**Location:** `_lym/discussion/decision-record.md:113` (D5)
**Issue:** Raised by interactions as F-I1, and by bound-breaker (F-d) and walkthrough (W1).
It is merged with F-I3, since the same definition decides whether the landing entry follows D3's removal rule.
D5 :122 lets a second landing proceed, but the record defines the state that makes the first run's `done` and pair hold safe only for one run.

### [BLOCKING] Synchronous pushes and batten's prime records bypass D5's records hold

**Location:** `_lym/discussion/decision-record.md:133` (D5)
**Issue:** Raised by interactions as F-I2. Merged with existing-paths F-EP2 and F-EP3, bound-breaker F-a and walkthrough W4: every counterexample breaks the same ordering promise at :135.
The worst path runs through the catch-up's synchronous `PushRun`. Worktree-Teardown's tips-on-origin branch has no parent-owed check, unlike the dobby's landed-pair teardown, so it removes the pair, and the pair teardown composite then writes `done` while the parent push may still be refused.

### [BLOCKING] The parent target's pusher pushes and writes done with no owner verdict

**Location:** `_lym/discussion/decision-record.md:281` (Constraints)
**Issue:** Raised by bound-breaker as F-e at MEDIUM.
It is held at BLOCKING because it departs from a listed PATTERN entry, PATTERN-owner-machine, whose text covers pushing a Finalize past its merge and the catch-up after an absence.
The beam case is covered by the beam pair's own reconcile. The prime's landing target has none.

### [MEDIUM] The refused mark and owed entries have no writer that clears them

**Location:** `_lym/discussion/decision-record.md:88` (D3)
**Issue:** Carries prior F4. Round 1 asked the record to say what clears the refusal, and the text now says clearing does so, but no path performs the clearing.
Raised by interactions (F-I5), bound-breaker (F-f) and walkthrough (W3).

### [MEDIUM] D9's base is read from a merge record this merge has not written

**Location:** `_lym/discussion/decision-record.md:210` (D9)
**Issue:** Carries prior F16. Its fix moved the base to the merge record's `task_head`, which the code writes only after the clean check the removal must run before.
Raised by walkthrough (W7), existing-paths (F-EP5) and bound-breaker (F-h).

## Not raised under the incremental rule

This round is incremental, so a new finding is raised only at BLOCKING on a changed hunk.
These lens findings are real but sit below that bar or on unchanged lines, and are recorded here for the next full review:

- F-EP4 (existing-paths, MEDIUM): moving the parent push to the pusher drops `pushParent`'s `archiveHeldParentMerge` and adopter-first repair (`finalize.go:974-985`), so an adopter-first landing never settles.
- F-c (bound-breaker, MEDIUM, unchanged :97): an owed push writes a failed-push record of an owed class today (`pushanchored.go`, `failedpush.go`), which the run reader reads as `PushOwed`, so "still writes no failed-push record" is not today's behaviour.
- F-I6 (interactions, LOW): nothing removes a dead driver's wait record, which D3's summary and all-clear read.
- F-I7 (interactions, LOW): D5 calls a diverged local parent "a refusal (D7)" without saying whether a read writes a failed-push record.
- W5 (walkthrough, LOW, unchanged :49): `RunCommitsAhead` does not count a plain code commit past the named tip, which D1's owed rule needs.
- W6 (walkthrough, LOW, unchanged :143): the pull-request check is placed "before it takes the parent lock", but `pullRequestLanding` decides the pull request under the lock (`finalize.go:407-425`).
- W8 (walkthrough, LOW): the driver's halt note for a stop that ends the wait duplicates the step's own note.
- F-g (bound-breaker, LOW): D7 does not pin `core.logAllRefUpdates`, whose default decides whether the code branch has a reflog.
- D5 :140-141's sentence on the past-the-merge read is garbled ("reads the local parent, and whenever … so a resume …"), though its intent matches prior F6's fix.

## Focus verdicts

- D5 with D6, landing against origin: the Goal, D5 and D6 now agree on Finalize's own reads, and PATTERN-owner-machine allows them. The Land row after it (F1) and the landing target's owner verdict (F4) break the outcome.
- D5 with D1, second landing and ordering: syncParent's ahead case is settled. The per-run state (F2) and the hold's bypass (F3) are not.
- Owed state: "owes a parent push" is defined once at :113-114 and read the same way by the three readers. The crash gap is covered at :125-126, and pushes-off at :127.
- Ledger signals: a wait record with nothing owed reaches the summary. The refused mark and stale entries never clear (F5).
- Network wait: the wait record is judged against `shedrun.DriverLock`, the step writes no note for a waited stop, and the bound names misconfigured hosts. This holds.
- Push paths: the by-kind rule covers `lym fabric commit`'s detached push, a beam parent gets one target, PushUntilLanded is replaced in the pusher, and the declared PATTERN-pair-remote text agrees with the kept synchronous pushes. This holds.
- Reset guard: the expiry defaults are pinned to git-gc(1), every refusal names a way forward, and the hand-over and destroy moves are settled. This holds, bar the LOW gap F-g.
- D9: the rules are evaluated by `git check-ignore --no-index` in a scratch repository and pinned to git docs and a test, the removal runs before the pre-merge clean check, and the bound states what it deletes. The base timing is not settled (F6).
- Acceptance criteria: each Goal fault, issue #238 included, has a red-first criterion. The Land-row stop is new and gets one through F1's fix.
