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

_No rounds yet._

## Question ledger

- Q1 How dobby replaces the waiting batten process: short passes with InnerRun's memory on disk and a `Yield` outcome (D4). Auto-pick: keeps batten's code and survives every `runproc.End` path.
- Q2 Where dobby is registered: under the beam's slug, `HubOwner` for the prime (D1). Auto-pick: the pair teardown ends a landed beam's dobby.
- Q3 Whether a stop verb is needed: no (D1). Auto-pick, YAGNI.
- Q4 Concurrency cap: `beam.yaml` `max_runs`, default 3, halted runs counted (D5). Auto-pick.
- Q5 What a new task's status is: `draft`, for every creating write and a promoted note (D5). Auto-pick, from the operator's 2026-10-18 "a new task is a draft by default" carried over to the status form.
- Q6 How a draft is released: `lyx board set-status <slug>` with no status, an existing verb (D5). Auto-pick.
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
