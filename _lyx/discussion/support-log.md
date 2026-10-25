# Support log — umbrella-daemon

## Interview

### Turn 1 — exploration (autonomous mode)

Read the board entry, the related entries (`umbrella-anchor`, `loom-beam-darns`, `landing-escalate-arbiter`, `batten-window-default`, `run-verbs-shared`, `discussion-scope-in-lenses`, `idle-notice-dead-waiter-cause`, `board-viewer`) and the docs of `battenshed`, `beamcli`, `orchengine`, `orchcli`, `boardengine`, `fswatch`, `runproc`, `tenderengine`, `sheddriver` and `arbiterstop`.
Found that InnerRun holds a lot of memory across checks in its producer, that shed has no yield outcome, that `runproc.End` on a slug ends every registered process of it, and that Finalize already has a file-level config notice.

### Turn 2 — parent decision (ly:orch, 2026-10-25)

`ly:orch` sent an operator decision: the section "The arbiter for every shed run" is in scope.
Move the arbiter seam into shed beside the tackler, wire it for every shed-driven recipe (batten and dobby included), put its keys in `shed.yaml` with per-module overrides, and give the arbiter a description of batten's rows.
Recorded as D8 and acknowledged to `ly:orch`.

### Turn 3 — design picks

Every remaining question was decided by the writer; see the question ledger.

### Turn 4 — lens review

Seven forks, one per lens; every fork returned a report.
Checked: before the edits, `lyx batten cancel` keeps the pair by default and clears the status, and `boardengine`'s store loads only version 1.

- interactions, BLOCKING, passes resume halted batten runs on every probe: fixed, D4 runs passes only while batten is `running`.
- interactions, BLOCKING, a cancelled task's empty status restarts it: fixed, cancel leaves `draft` (D5).
- interactions, MEDIUM, dobby under the beam slug ended by the beam's own landing run: fixed, D1 states the bound.
- interactions, MEDIUM, refusal latch on `revised`: fixed, backoff retry plus entry change (D5).
- interactions, MEDIUM, row notices ahead of the arbiter: fixed, D8 holds them until the hand-back.
- interactions, MEDIUM, `ResumeInWindow` callers: fixed, D4 has adopt and beam land start a detached first pass.
- interactions, MEDIUM, landing count across a dobby restart: fixed, D10 counts from batten history keyed by run.
- interactions, MEDIUM, two prime dobbies: fixed, D1 and D2 tie the prime dobby to the prime owner.
- interactions, LOW, watched pid after a re-exec on Windows: fixed, each pass writes the marker with the current dobby pid (D4).
- interactions, LOW, batch upsert and merge skip the draft default: fixed (D5).
- existing-paths, BLOCKING, Open-Window and origin reconcile on every pass: fixed, continue passes skip both (D4).
- existing-paths, BLOCKING, passes revive a run being cancelled or promoted: fixed, no pass while the cancel marker exists, which promote's end also takes (D4).
- existing-paths, MEDIUM, `pairteardown` reads the gap between passes as a gone batten: fixed (D4).
- existing-paths, MEDIUM, `beam land` and window wording in the orch role: fixed (D4, D6).
- existing-paths, MEDIUM, draft guarded only in dobby: fixed, start guard refuses a draft (D5).
- existing-paths, MEDIUM, row notices ahead of the arbiter: merged with the interactions finding, fixed.
- existing-paths, LOW, beam slug owner pause ends dobby: merged with interactions, fixed.
- existing-paths, LOW, watched marker written with the pass's pid: fixed (D4).
- bound-breaker, BLOCKING, passes trigger themselves through batten's status file: fixed, the yield persists nothing and a write while a pass is in flight triggers none (D4).
- bound-breaker, BLOCKING, cancel restarts: merged, fixed.
- bound-breaker, MEDIUM, refusal never retried: merged, fixed.
- bound-breaker, MEDIUM, yield as an interrupted step: fixed, a `Yield` shed outcome (D4).
- bound-breaker, MEDIUM, darn's arbiter values change: fixed, the migration copies loom's keys into `darn.yaml` (D8).
- bound-breaker, MEDIUM, row notices ahead of the arbiter: merged, fixed.
- bound-breaker, MEDIUM, wait latencies grow between passes: fixed, D4 states the one-probe bound.
- bound-breaker, MEDIUM, tidy agent's parent is dobby, and no timeout: fixed, its parent is the nearest orch and `board_tidy_timeout_min` bounds it (D10).
- bound-breaker, MEDIUM, no-migration evidence goes stale before landing: fixed, a one-time migration to `draft` (D5).
- bound-breaker, MEDIUM, two machines start one task: merged with the prime-owner finding, fixed.
- bound-breaker, LOW, remote releases unseen until a pull: fixed, D5 states it.
- bound-breaker, LOW, beam land through `ResumeInWindow`: merged, fixed.
- bound-breaker, LOW, an older binary skips the draft default: rejected, the migration covers what such a binary wrote before the key exists, and an owner machine runs its deployed binary.
- bound-breaker, LOW, config deletions unspecified: fixed, `absent` (D11).
- bound-breaker, LOW, a failed `lyx beam start` only warns: fixed, the Warn names `lyx beam start`; the orch's start still must not fail on it.
- walkthrough, BLOCKING, no yield mechanism: merged with bound-breaker, fixed.
- walkthrough, BLOCKING, per-episode spawn confirmation leaves a dead driver: fixed, confirmation stays per pass with a respawn cap (D4, D9).
- walkthrough, MEDIUM, where the runs come from: fixed, local batten status files (D3).
- walkthrough, MEDIUM, self-triggered loop: merged, fixed.
- walkthrough, MEDIUM, hand-run start of a draft: merged, fixed; the PATTERN text allows the hand pass.
- walkthrough, MEDIUM, refusal retry key: merged, fixed.
- walkthrough, MEDIUM, uncapped api-error revive: fixed, notice on the third revived episode in a row (D9).
- walkthrough, MEDIUM, adopt leaves no dobby: fixed, adopt ends with `lyx beam start` and a dobby owned elsewhere exits (D1, D2).
- walkthrough, LOW, `max_runs` counting: fixed, halted runs count (D5).
- walkthrough, LOW, pause during Land: fixed, InnerRun yields for a done child under a pause (D12).
- walkthrough, LOW, tidy across a re-exec: fixed (D10).
- goal-scope, MEDIUM, dobby's failures skip the arbiter: rejected, the arbiter judges a shed stop from its run's records, and dobby's failures are refused starts and failing passes; kept as an auto-mode assumption for the parent review to see.
- goal-scope, MEDIUM, pause during Land: merged, fixed.
- goal-scope, LOW, D11 departs from the entry's wording: fixed, D11 says so.
- goal-scope, LOW, background-fork strand after a tender resume: fixed, out of scope in Scope.
- goal-scope, LOW, stale orch rule: fixed, Scope says the dropped notice leaves it no trigger.
- goal-scope, LOW, `--window` retirement unlicensed: fixed, D4 names it as an addition.
- goal-scope, LOW, refusal latch: merged, fixed.
- board-fit, MEDIUM, overlap with `discussion-scope-in-lenses`: fixed, a Constraint.
- board-fit, MEDIUM, `landing-escalate-arbiter` unnamed: fixed, out of scope in Scope.
- board-fit, MEDIUM, process volume: fixed, D4's per-run bound and the risk names `spawn-audit-log`.
- board-fit, MEDIUM, `ResumeInWindow` callers: merged, fixed.
- board-fit, LOW, `loom-beam-darns` downstream: fixed, named in Scope.
- board-fit, LOW, `creel-agents` overlap: rejected, a note with no design yet, and D9 keeps today's notices.
- board-fit, LOW, README mark against `board-viewer`: rejected, that note's restructure carries whatever marks the README has.
- board-fit, LOW, `idle-notice-dead-waiter-cause` touched: rejected, its Warn is a once-per-episode rule D4's memory record already covers.
- leanness, MEDIUM, `max_runs` assumption claims an orch cap: fixed, bullet cut.
- leanness, LOW, "dobby included" stated twice: fixed, D7's rationale cut.
- leanness, LOW, statusless tasks stated three times: fixed, the migration replaced the claim.
- leanness, LOW, bare PATTERN mappings in Constraints: fixed, cut.
- leanness, LOW, Notes for the plan writer repeats relevant.md: fixed, section dropped.
- leanness, LOW, note removal with no decision: fixed, in D4.
- leanness, LOW, nonstandard ledger tag: fixed.

## Rejected alternatives

- Dobby hosting each run's batten driver in-process as a goroutine: it keeps InnerRun's memory, but `runproc.End` on a run's slug, which cancel, promote, the owner pause and teardown all call, would end dobby and every run it hosts.
- Dobby supervising one long-lived batten process per run: the least change, but the operator agreed that the daemon replaces the waiting process per run.
- Ending a pass by a plain process exit: the Go driver reads that as an interrupted step and spends its repair cap, so the yield is an outcome of its own.
- Dobby registering under `HubOwner` for every beam: the pair teardown of a landed beam would then not end it.
- A `lyx beam stop` verb: the teardown ends a landed beam's dobby, the binary re-exec covers deploys, and tender is the brake; nothing asked for a stop.
- Raising the board store's version for the draft migration: an older binary on another machine would refuse to load the board.
- The key-level config check in dobby: only Finalize holds the base, tip and parent tip under the parent lock.
- Sending dobby's own failures to the arbiter: they are refused starts and failing passes, not shed stops, and the arbiter judges a stop from its run's records.
- Moving batten's logic into dobby (the entry's optional step 2): batten stays the lifecycle owner, as the operator decided.

## Review rounds

### Round 1

Every finding's premise was checked against the code and held; all were fixed.
The fixer's self-check forks then corrected the fixes to F4, F8, F9 and F11, and added the owed full pass's clearing, the `--continue-pass` refusal's handling in dobby and the orch stencil's resume wording.

- F1, BLOCKING, continue passes skip the reconcile on a false premise: fixed, dobby holds the sleep watch and owes every run a full pass after a wake; D3's owner-check claim rewritten (D3, D4).
- F2, BLOCKING, pair teardown clears a removed task's status: fixed, the claim settlement leaves `draft` (D5).
- F3, BLOCKING, memory inventory deferred: fixed, D4 lists every persisted field and its rule; the Open risk is gone and the notice-marker Constraint names the in-memory latches.
- F4, MEDIUM, a pass stays in flight through in-process waits: fixed differently, the spawn hold yields, while the Go driver's pause hold stays, since it sits inside stop handling and ending the pass there would strand the stop; every other wait has its bound stated, and D12 says how the lift reaches a pass in flight.
- F5, MEDIUM, `lyx reed down` ends a beam's dobby: fixed, `EndBeforeSessionDown` spares a `RoleDobby` record (D1).
- F6, MEDIUM, `--continue-pass` unbounded: fixed, the pass refuses it off Run-Shed or off its dobby's record (D4).
- F7, MEDIUM, who resumes a halted run: fixed, a hand pass by the operator or the orch (Goal, D4, D6).
- F8, LOW, respawn cap spent in gaps: fixed differently, the count resets on a write of the child's status file since the last respawn, since a reset on liveness alone lets a driver that dies minutes after each respawn escape the cap (D4).
- F9, LOW, a third live-batten reader: fixed, the rule sits on `battenAlive` (D4).
- F10, LOW, darn's copy covers the hub layer only: fixed, D8 states that only hub-layer values carry over.
- F11, LOW, landings outside dobby passes: fixed, counted from batten history on each tick (D10).
- F12, LOW, older binaries create statusless tasks: fixed, an accepted bound in D5; this replaces Turn 4's rejection, which reasoned about the starter's binary.
- F13, LOW, re-exec versus start: fixed, a re-exec is not a start (D4).
- F14, LOW, two dedup keys for dobby's notices: fixed, D7 uses D4's and D5's keys and says when each clears.
- F15, LOW, backoff with only halted runs: fixed, D13 defines in flight for the backoff.
- F16, LOW, late fswatch event after a pass: fixed, a write no later than the last pass exit triggers nothing (D4).
- F17, LOW, D11 rationale-by-rejection: fixed, sentence deleted; this supersedes Turn 4's goal-scope fix that added it.

### Round 2

Every finding's premise was checked against the code and held; all were fixed.
The fixer's self-check forks then widened F1 to every owned-elsewhere refusal, kept the full pass owed while a run is marked owned elsewhere, kept such a run out of `max_runs` and the backoff, retried a failed owner pause, limited the Open-Window re-run in F6's owed episode, and added the beam-side refusal to F7's.

- F1, BLOCKING, a takeover found by a full pass loops as a failing pass: fixed, the owned-elsewhere refusal has its own exit code, and dobby pauses the run under the owner, releases its parent lock, sends the owner notice once and passes it no more until a full pass at the lift or from `lyx adopt` exits 0 (D3, D4, D7).
- F2, BLOCKING, the spawn hold yields before the owner re-check: fixed, the hold runs the owner check on a due probe before yielding (D4).
- F3, BLOCKING, continue passes skip the origin verdict without a PATTERN change: fixed, a `PATTERN-owner-machine` change says a continue pass is not a resume (D4, PATTERN changes).
- F4, MEDIUM, a pass's one check is unnamed: fixed, it is `check()`'s full body, and `ownerSleep` is replaced by dobby's sleeps and the owed full pass (D4).
- F5, MEDIUM, the new counters have no home: fixed, an inventory bullet names them (D4).
- F6, MEDIUM, an unreachable origin clears an owed full pass: fixed, the pass reports it and the full pass stays owed until one reaches origin (D4).
- F7, MEDIUM, an adopted member of a beam owned elsewhere has no dobby: fixed, `lyx adopt` of such a member refuses unless the same call adopts the beam (D2); picked over driving it from an ancestor's dobby, which would break D3's one-parent rule.
- F8, MEDIUM, a note upserted to a task stays statusless: fixed, it gets `draft`, and a null status upsert or nil merge `set_status` is a release (D5).
- F9, LOW, D4's list of in-pass waits is incomplete: fixed, Land's deploy and the spawn's `lyx loom start` bootstrap are named (D4).
- F10, LOW, cleanup can remove a landing before dobby counts it: fixed, an accepted bound (D10).

### Round 3

Every finding's premise was checked against the code and held; all were fixed.
The fixer's self-check forks then let continue passes run under a covering pause, so an idle owner's parent lock is still released, moved D2's refusals ahead of the owner write with one combined way forward, kept a start refused for a pause out of D5's refused-start path, and aligned D3, D13 and the criteria.

- F1, BLOCKING, a pass under a covering pause is refused at pre-run, not yielded: fixed differently, `--continue-pass` skips the pre-run pause refusal so continue passes run under a pause, dobby runs no full pass of a covered run bar the lift's, and a full pass refused for a pause has its own exit code read as covered (D4, D5, D7, D13).
- F2, MEDIUM, the owed full pass of an unreachable-origin resume is invisible to a hand pass: fixed, the pass writes a per-run origin-owed marker that dobby reads, replacing the output line (D4).
- F3, MEDIUM, a failed owner pause is retried on a stale verdict: fixed, dobby runs a full pass on the probe tick instead of a blind retry (D4).
- F4, MEDIUM, the adopt refusal omits a beam whose parent is owned elsewhere: fixed, the refusal covers a task or a beam, is decided before the owner write against the entries that passed step 1, and names one call that meets both refusals (D2, Q21).
- F5, MEDIUM, the release command fails as written: fixed, it names the explicit null payload (D5, Q6).
- F6, MEDIUM, the draft-giving writes are not a closed set: fixed differently, one rule at the store's commit point, with empty meaning nil or an empty string, which also spares a released task's later edits, and no lyx code path writes a null task status (D5).

## Question ledger

- Q1 How dobby replaces the waiting batten process: short passes with InnerRun's memory on disk and a `Yield` outcome (D4). Auto-pick: keeps batten's code and survives every `runproc.End` path.
- Q2 Where dobby is registered: under the beam's slug, `HubOwner` for the prime (D1). Auto-pick: the pair teardown ends a landed beam's dobby.
- Q3 Whether a stop verb is needed: no (D1). Auto-pick, YAGNI.
- Q4 Concurrency cap: `beam.yaml` `max_runs`, default 3, halted runs counted (D5). Auto-pick.
- Q5 What a new task's status is: `draft`, for every creating write and a promoted note (D5). Auto-pick, from the operator's 2026-10-18 "a new task is a draft by default" carried over to the status form.
- Q6 How a draft is released: the existing verb with an explicit null status, `lyx board set-status '{"slug":"<slug>","status":null}'`, since it refuses an absent `status` key (D5). Auto-pick, over changing `set-status` to accept a missing status, which would let a typo clear a status.
- Q7 Draft mark in the README: `draft` before the title (D5). Auto-pick.
- Q8 Which batten notices stay (D9). Auto-pick, keeping every notice that needs judgement.
- Q9 Where the arbiter code moves: a shed-layer package batten imports without tree-sitter (D8). Auto-pick within the parent's decision.
- Q10 Batten's arbiter eligibility: tackler escalations except hand-backs (D8). Auto-pick.
- Q11 Where the key-level config check runs: Finalize (D11). Auto-pick.
- Q12 What "dobby included" means for the arbiter (Auto-mode assumptions). Auto-pick.
- Q13 Removing the hub-wide orch rule on the board tidy: left to the parent through `lyx orch rules` (Scope). Auto-pick: a Bolt rules write is not a run's to make.
- Q14 What a cancel leaves: `draft` (D5). Auto-pick, so a cancel is never undone by dobby.
- Q15 Existing statusless tasks: a one-time migration to `draft` keyed on a board key (D5). Auto-pick, so nothing made before this lands starts unreleased.
- Q16 Where dobby finds its runs: local batten status files (D3). Auto-pick: the board loses a run once Land prunes it.
- Q17 How a wake reaches the runs: dobby's own sleep watch owes every run a full pass (D4). Auto-pick: an awake clock reading is only meaningful inside the process that holds it, and dobby lives across the passes.
- Q18 What `lyx reed down` does to a beam's dobby: spares it (D1). Auto-pick: dobby runs outside the session, and the operator decided a run's parent always has a running dobby.
- Q19 Which layers D8's darn copy covers: the hub layer only (D8). Auto-pick: the other layers belong to one worktree or one task, and a task's `run_settings` names the module it targets.
- Q20 Who resumes a halted or paused run: a hand pass by the operator, or the orch on the operator's word or after judging the stop (D4). Auto-pick.
- Q21 Who drives a run whose parent another machine owns, a member task's run or a beam's landing run: nobody, so `lyx adopt` refuses to split a task or beam from its parent, and a beam from its members, either way (D2). Auto-pick: driving it from an ancestor's dobby would break D3's one-parent rule.
- Q22 How continue passes fit `PATTERN-owner-machine`: a PATTERN change saying a continue pass is not a resume, over dropping the reconcile skip (D4). Auto-pick: a reconcile per probe tick per run would put a network round trip on every pass.
- Q23 What a full pass refused as owned elsewhere leaves: a mark in dobby's state record, an owed full pass and today's owner pause, passed again only at the pause's lift, or on each probe tick while the owner pause failed (D4). Auto-pick: the lift is the one event an adopt back and an operator's resume share, and a full pass re-judges the owner where a blind pause retry would not.
