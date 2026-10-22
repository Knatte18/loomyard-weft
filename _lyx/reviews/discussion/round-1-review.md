---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    origin: lens:existing-paths
    location: _lyx/discussion/decision-record.md D2 and D3 step 2
    summary: The run-scope origin reconcile errors when the run has no pair, and D2 refuses every reconcile error but an unreachable origin, so the pair-less blocked run of 2026-10-20, the entry's headline case and acceptance criterion 1, can never be cancelled.
    fix: In D3 step 2 and D2, state that with no pair worktree present (no status, or a status at Worktree-Create with no pair, or any state whose pair is gone) the verb runs no run-scope reconcile and instead reads the owner from the Bolt it just pulled, as batten's start check `ownerPreflight` does (internal/battencli/owner.go:194-205); cite internal/fabricengine/originreconcile.go:574-576 as the evidence, and add a D2 bullet for what a failed Bolt pull then does.
  - id: F2
    severity: BLOCKING
    class: design
    origin: handler
    location: _lyx/discussion/decision-record.md D5
    summary: RemovePair takes a snapshot only when Request.Local and Request.Snapshot are both set, so D5's Force plus Snapshot with the ordinary origin steps takes no snapshot and force-discards the pair's uncommitted paths, breaking D5's bound and acceptance criterion 4.
    fix: Rewrite D5's first paragraph to name the actual mechanism, either a pairteardown change that takes the snapshot before an origin-mode removal (and add that change to the Scope's pairteardown line), or the local-mode removal with Snapshot, stating what that mode then skips (archive tag push, remote branch removal); cite internal/pairteardown/teardown.go:347 and internal/fabricengine/remove.go:126-128 as the evidence, and keep D5's Bound line true for the mechanism chosen.
  - id: F3
    severity: BLOCKING
    class: design
    origin: handler
    location: _lyx/discussion/decision-record.md D5 and Acceptance criteria (rerun criterion)
    summary: Remove deletes the origin task branch only when its work is landed, so a cancelled run's pushed branch stays on origin, and a fresh Add's probeWarpLeftover then refuses it; neither the --remove rerun nor the kept-pair rerun after lyx fabric remove starts fresh.
    fix: In D5, decide and state what the cancel does with the task branch on origin (and with a local branch the destructive gate keeps), for example delete it under the cancel's own authority after the snapshot or archive tag preserves it, or name the extra step a rerun needs; cite internal/fabricengine/remove.go:148 and :683 and internal/fabricengine/remoteleftover.go:157-161, and correct the rerun acceptance criterion and the notice wording to match.
  - id: F4
    severity: BLOCKING
    class: consistency
    origin: handler
    location: _lyx/discussion/decision-record.md D4 (steps 1 to 3 and the paragraph beginning Steps 3 to 7)
    summary: D4 runs runproc.End in step 1 outside the new pairteardown phase, which covers only steps 3 to 7, while the declared PATTERN-pair-teardown text says a batten cancel's phase itself ends the owned processes and strands first.
    fix: Either make the cancel phase in internal/pairteardown own D4 steps 1 to 7 (the write-locked End, batten's lock take, strand stops, loop end, run-lock wait, session end and mark withdrawal), with only the no-pair case's End left to the verb, or reword the declared PATTERN text in the PATTERN changes section so it says the verb ends the owned processes before the phase; keep D4 and the PATTERN text saying the same thing.
  - id: F5
    severity: MEDIUM
    class: design
    origin: handler
    location: _lyx/discussion/decision-record.md D3 step 2
    summary: The run-scope reconcile is a moving scope that never asks LivePair about its own pair, and D3 runs it before D4 ends the live run, so it can fast-forward the task worktree or replay the pair's records branch under a working agent and step.
    fix: In D3 step 2, state that the cancel runs the reconcile with ReconcileScope.JudgeOnly for its verdicts (internal/fabricengine/originreconcile.go:34-37), or state that it moves nothing of the slug's own pair before step 4; cite internal/fabricengine/doc.go:83-87.
  - id: F6
    severity: MEDIUM
    class: decision
    origin: lens:walkthrough
    location: _lyx/discussion/decision-record.md D2
    summary: The run-scope reconcile reports the run's own entry ended (landed, or removed from the board) without a teardown, and D2 names no path for that verdict, although an entry re-scoped off the board is the board entry's own 2026-10-18 case.
    fix: Add to D2 what an ended verdict does - for a landed reading, the reached-parent refusal; for an entry removed from the board, go on with the cancel and leave D7's clear to find no entry - and cite internal/fabricengine/doc.go:76-80.
  - id: F7
    severity: MEDIUM
    class: design
    origin: handler
    location: _lyx/discussion/decision-record.md D2 (the unreachable-origin paragraph and its Bound)
    summary: The unreachable-origin path states no bound on what it skips - a stale local Bolt can read a run adopted by another machine as owned here, so the verb removes records and clears a claim that later sync onto the owner's run, and a stopped branch goes unread - and with --remove every offline cancel stops at Remove's archive push, so D2's cancels-with-a-Warn criterion never holds for --remove.
    fix: In D2, state the bound of the offline path - what it can no longer judge (owner adopted since the last pull, stopped branches) and why that is accepted or refused - and state that with --remove an unreachable origin refuses before anything is ended (or switches to the local removal), naming the way forward; update the matching acceptance criterion.
  - id: F8
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    location: _lyx/discussion/decision-record.md D3 (Steps 4 to 8 run under batten's locks) and the acceptance criterion on a run started during a cancel
    summary: A batten start writes its seed and registers its process in armSeed before taking any batten lock, so a lyx batten run arriving after step 6 reseeds the slug while the cancel holds the locks, and a resume in the gap between D4 step 1's End and step 2's lock take can still run.
    fix: In D3, state what bars a start during the cancel - for example the cancel holds batten's prime run lock from before step 4's End, or the start path checks a cancel marker before armSeed - and cite internal/battencli/arm.go:377-387; correct the criterion if the guard is narrower.
  - id: F9
    severity: MEDIUM
    class: design
    origin: lens:existing-paths
    location: _lyx/discussion/decision-record.md D2 (Nothing is left to cancel) and D3 (interrupted cancel finishes on a re-run)
    summary: A --remove cancel interrupted after step 7 leaves no records, no run-form status, no pair and no process, so the re-run refuses as nothing to cancel and step 8 never lifts the slug pause, spawns the records push or posts the notice.
    fix: Add a live slug-scope pause entry and an unpushed records commit to D2's list of what counts as something left to cancel, or order step 8's lift and push before step 7's clear, so D3's re-run claim holds.
  - id: F10
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    location: _lyx/discussion/decision-record.md Goal (Windows sentence) and D4 survivor paragraph
    summary: On Windows runproc.End returns nil and signals nothing, so D4's survivors-after-End rule detects nothing there; a live run is refused only after step 2 spends the two-minute run-lock bound, and a live process holding no lock passes.
    fix: In D4, state the Windows check - read the slug's live registered processes with runproc.List before step 1 and refuse at once, naming each pid with the taskkill line - and cite internal/runproc/end.go:110-112.
  - id: F11
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    location: _lyx/discussion/decision-record.md D2 (hub pause added after this check) and D3/D4 step 2 (status lock held through step 8)
    summary: The verb holds batten's prime status lock exclusively while step 5's origin calls wait at the network gate, and hibernate takes the hub pause and then blocks on that same status lock, so a --remove cancel and lyx hibernate request wait on each other unbounded; every blocking status reader waits out the whole cancel too.
    fix: In D4 step 2 and D2, either hold only the run and driver locks across the network steps and take the status lock only around the status reads and removal, or exempt the verb's network gate for its duration as lyx fabric remove does (internal/fabriccli/doc.go:56); state which readers block and for how long.
  - id: F12
    severity: MEDIUM
    class: design
    origin: lens:existing-paths
    location: _lyx/discussion/decision-record.md D6
    summary: cleanupSlug takes batten's run lock by try-acquire and its status lock by blocking acquire on fresh flock handles, which conflict with the verb's own held locks in the same process, so reused under D4's locks it skips the slug or blocks for good.
    fix: In D6, state that the reused removal step takes no lock of its own and runs under the caller's held locks, the split covering the lock-taking as well as the done judgement; cite internal/battencli/cleanup.go:187-206 and internal/lock/lock.go:27-38, and align the plan-writer note.
  - id: F13
    severity: LOW
    class: consistency
    origin: handler
    location: _lyx/discussion/decision-record.md D4 steps 2 and 5 and survivor paragraph
    summary: Step 2's run lock is batten's prime-side run lock while step 5's run-lock wait is the pair's own run lock, and the survivor rule's driver lock or run lock names neither, so the sequence reads as taking and then waiting for one lock.
    fix: Name each lock by owner in D4 - batten's prime-side driver, run and status locks in step 2 and in D3 step 4; the pair's run lock in step 5; and in the survivor paragraph which driver lock and run lock are meant.
  - id: F14
    severity: LOW
    class: consistency
    origin: lens:bound-breaker
    location: _lyx/discussion/decision-record.md D7
    summary: D7 says the verb clears the claim in one Bolt commit and push by itself, but a board write saves locally and spawns a detached sync that does the push, so the verb's envelope cannot report the push outcome D2 and D9 rely on.
    fix: Reword D7 to say the verb writes the status through boardengine and the board's detached sync pushes it, with a failed push surfacing through that sync's failed-push record; drop any envelope claim about the board push.
  - id: F15
    severity: LOW
    class: consistency
    origin: handler
    location: _lyx/discussion/decision-record.md Notes for the plan writer, third bullet
    summary: The note places batten's tackler in-flight marker under shedrun.TacklerDir, but it is tackler-inflight.json under ScratchDir via shedrun.TacklerMarker.
    fix: Replace the bullet's path with shedrun.TacklerMarker, under the run's ScratchDir, read before D3 step 6 empties the scratch.
---

### [BLOCKING] A pair-less run cannot be cancelled

**Location:** `_lyx/discussion/decision-record.md` D2 and D3 step 2

**Issue:** `internal/fabricengine/originreconcile.go:574-576` returns `no pair worktree` for a run scope whose pair is absent.
D2 turns every reconcile error but an unreachable origin into a refusal, so the 2026-10-20 case (blocked at Worktree-Create, no pair) refuses.
Batten's own pre-run avoids this by running the start check instead (`internal/battencli/owner.go:194-205`).

### [BLOCKING] `--remove` takes no snapshot in origin mode

**Location:** `_lyx/discussion/decision-record.md` D5

**Issue:** `RemovePair` snapshots only `if req.Local && req.Snapshot` (`internal/pairteardown/teardown.go:347`), and the package doc puts Snapshot in local mode.
D5 asks for Snapshot with the ordinary origin steps, where `Remove` with force removes uncommitted changes outright.
D5's bound and acceptance criterion 4 rest on a false premise, and the Scope names no change that closes it.

### [BLOCKING] A rerun after a cancel does not start fresh

**Location:** `_lyx/discussion/decision-record.md` D5 and the rerun acceptance criterion

**Issue:** `Remove` deletes the task branch on origin only when its work is landed (`remove.go:148`, `:683`), which a cancelled run's work never is.
`probeWarpLeftover` refuses a pair that is not live when the origin tip is not an ancestor of the HEAD it forks from (`remoteleftover.go:157-161`).
So both rerun paths the record promises meet a refusal at Worktree-Create, and the record never decides the origin branch's fate.

### [BLOCKING] The owned-process end sits outside the declared cancel phase

**Location:** `_lyx/discussion/decision-record.md` D4

**Issue:** The declared PATTERN-pair-teardown text says "a batten cancel's phase instead ends the owned processes and strands first and waits for the run lock after".
D4 makes `runproc.End` step 1, run by the verb under the write locks, and only steps 3 to 7 the phase.
This departs from the listed text, which binds as a constraint.

### [MEDIUM] The reconcile moves a live pair before the end

**Location:** `_lyx/discussion/decision-record.md` D3 step 2

**Issue:** Any scope but judge-only moves branches, and the run scope never asks `LivePair` about its own pair, because the run's own driver is normally the caller (`internal/fabricengine/doc.go:83-87`).
In a cancel the run is still live at step 2.

### [MEDIUM] D2 leaves the ended verdict undecided

**Location:** `_lyx/discussion/decision-record.md` D2

**Issue:** The run scope reports its own run ended with no teardown (`internal/fabricengine/doc.go:80`), and D2's refusals cover every other verdict but this one.

### [MEDIUM] The unreachable-origin path has no stated bound

**Location:** `_lyx/discussion/decision-record.md` D2

**Issue:** Offline, the owner comes from the local Bolt, which can be stale after another machine's adopt.
The cancel then removes the slug's records on the shared prime records branch and clears the claim, and both sync onto the owner's live run later.
A stopped branch also goes unread.
With `--remove`, `Remove`'s archive push fails first and "returns its error with everything still in place", so an offline `--remove` always stops at D5's refusal.

### [MEDIUM] A start can seed during the cancel

**Location:** `_lyx/discussion/decision-record.md` D3

**Issue:** `prepare` runs `armSeed`, which writes the seed and registers the process before any batten lock is taken (`internal/battencli/arm.go:377-387`).
A start arriving after step 6 leaves a fresh seed behind the cancel.
A resume in the gap between D4 steps 1 and 2 can still act.

### [MEDIUM] An interrupted `--remove` cancel cannot finish on a re-run

**Location:** `_lyx/discussion/decision-record.md` D2 and D3

**Issue:** After step 7 nothing D2 counts is left, so the re-run refuses.
The slug pause D8 says would hold a later run stays live, and the push and the notice never happen.

### [MEDIUM] The Windows refusal comes only after the full wait

**Location:** `_lyx/discussion/decision-record.md` Goal and D4

**Issue:** `runproc.End` returns nil on Windows (`internal/runproc/end.go:110-112`), so no survivor is reported.
A live batten is refused only when step 2 spends its two-minute bound, the "looked like a hang" symptom the entry names.
A live process holding no lock is not refused at all.

### [MEDIUM] The status lock held across the network gate deadlocks with hibernate

**Location:** `_lyx/discussion/decision-record.md` D2, D3 and D4 step 2

**Issue:** Remote git calls wait at the network gate while a pause covers the process (`internal/gitexec/doc.go:23`).
Hibernate puts the hub pause in place, then reads each run's status under a blocking read lock (`internal/hibernatecli/deps.go:127`).
D2's line "as every prime verb's do" does not hold for a verb holding a run's status lock; `lyx fabric remove` exempts its own gate for this reason.

### [MEDIUM] Reusing cleanup's step conflicts with the held locks

**Location:** `_lyx/discussion/decision-record.md` D6

**Issue:** `cleanupSlug` acquires both locks itself on new `gofrs/flock` handles, and those conflict within one process.

### [LOW] Two run locks share one name in D4

**Location:** `_lyx/discussion/decision-record.md` D4

**Issue:** Once each lock is read by its owner, the sequence holds together, and D3 step 4 matches step 2.
The wording does not separate the two locks, though, and a plan writer could merge them.

### [LOW] D7 misplaces the board push

**Location:** `_lyx/discussion/decision-record.md` D7

**Issue:** `boardCriticalSection` saves locally and spawns a detached sync (`internal/boardengine/board.go:53`).

### [LOW] Wrong tackler marker path in the note

**Location:** `_lyx/discussion/decision-record.md` Notes for the plan writer

**Issue:** `shedrun.TacklerMarker` returns `tackler-inflight.json` under `ScratchDir` (`internal/shedrun/paths.go:282-284`).
`TacklerDir` is the durable attempts directory.

## Rejected

- Arbiter strand of batten's prime run left alive (bound-breaker, LOW): batten's prime-side driver uses only the tackler marker (`internal/battencli/drive.go:260`), and `wire.go:688` reads it as an arbiter marker for the child seam only; the fork's own evidence leaves it unconfirmed.
- Support log Interview Turn 3 still records refusal on an unreachable origin: the Interview is chronological, and Turn 5 and ledger Q10 both record the revision, so a reader is not misled about the final design.
- Open risks (`index.lock` and older-binary processes) as deferred research: each states its consequence and its existing remedy (the failing commit names the lock; `lyx tender reap`), and neither hands downstream a halt or an unanswered fact.
- `Lift` sending no resume line with no sessions: researched and true (`tenderengine/resume.go:140-200`, and `PushPair` leaves out a gone pair), so it is no deferred fact.
- Relocation of Scope, Constraints, Auto-mode assumptions or Open risks to the support log: each carries a requirement or bound the plan writer needs, and none holds deliberation narrative.
- D11's hand-back row and D1's orch reach: each states its bound (the row only hands back, only a prefix-setting producer reaches it; the verb acts on one named slug from the prime).
- D4's skipped quiet wait and refusal probe: the refusal probe's checks rerun inside `RemovePair` under `--remove`, and without it there is no removal to protect; D5 names the after-end refusal's way forward.
