# Support log — twb-d0-hf

## Interview

### Turn 1 — board and code read

Read the board entry, `pause-background-work`, `orch-split`, the tender packages' docs and CLI, `pauserecord`'s doc and sleep watch, `orchengine`'s pause handling, the batten recipe, the shortcuts file and the overview's tender and shortcut rows.
Established that no hibernate code exists at HEAD, that `lyx pause` and `lyx resume` are already shortcuts, and that no machine-wide hub registry exists.

### Turn 2 — design settled by auto-pick

The run is autonomous, so every question was settled by auto-pick; see the Question ledger.

### Turn 3 — lens review

Six lens forks reviewed the record; every fork returned a report. Duplicates are merged onto the first line that names them.

- premises, MEDIUM, a hub pause during Land halts the landing, so the land-lock condition cannot simply wait at confirmation: fixed differently: the proposal to wait for the lock to be free and check it again at confirmation leaves a window between check and pause; instead D6 takes each hub's land lock before pausing and holds it to the end, with an Open risk for a Land that starts meanwhile. Merges pattern-fit LOW "land-lock owner".
- premises, MEDIUM, the land lock is not in tender status and is a try-acquire only: fixed as proposed: D6 names it a separate seam through its owner, and a Note names `battenshed.LandLock`.
- premises, MEDIUM, termwindow's WSL test is inline and unexported: fixed as proposed: Scope and D10 export it from `termwindow`.
- premises, LOW, `standalonestate` gives no machine-level dir: fixed as proposed: D3 exports the state-home root from `standalonestate`. Merges pattern-fit MEDIUM "standalonestate.Derive is cliwire's": D3 keeps `Derive` cliwire's alone, which `internal/cliwire/callerset_enforcement_test.go` enforces for `Derive` only, and the old Note is removed.
- premises, LOW, caller `hibernate` is not a strand name: fixed as proposed: D6 calls it a record label that is never a strand name.
- premises, LOW, `QueueNotice` needs the orch's paths: fixed as proposed: Constraints lists the orch notice paths as told.
- board-fit, MEDIUM, the orch's own turn in confirmation: fixed as proposed: D7 says the orch counts and its watcher's grace-end interrupt ends its turn, and `pause-background-work` owns any change.
- board-fit, MEDIUM, halted batten under `shed-tackler`: fixed differently: neither proposed "stop with no tackler in flight" nor firing only on landing with no answer for a halt; D5 fires only on landing and queues one orch notice on a halt or failure, so the request never waits silently. Merges goal-scope MEDIUM "halt is not landing".
- board-fit, LOW, `creel` folds the notice queue: fixed as proposed: Note added.
- board-fit, LOW, `spawn-audit-log` plans one launch path: fixed as proposed: the same Note names it.
- board-fit, LOW, runs started after the request: fixed as proposed: D5 says they are held by the pause, not waited for.
- goal-scope, MEDIUM, `pause-background-work` is a precondition, not only a risk: fixed as proposed: a Dependency line in Constraints with the build order; the Open risk is removed.
- goal-scope, LOW, no refusal-spec rows: fixed as proposed: Scope adds a `## hibernate` section in `refusal-spec.md`, and the Constraints line names it.
- goal-scope, LOW, the scan may pause fixture hubs: fixed as proposed: D4 skips hubs under the lyx temp root.
- goal-scope, LOW, the orch learns nothing on `done` or `cancelled`: fixed as proposed: D8 states it.
- leanness, MEDIUM, D11 and D14 restate Constraints, and D5 and D8 restate PATTERN lines: fixed as proposed: D11 (spawn logging) and D14 (sandbox) are dropped as Decisions and live in Constraints; D5 no longer restates PATTERN-daemon-wakeups.
- leanness, LOW, Auto-mode assumptions restate decisions: fixed as proposed: each is shortened to its point and D-number.
- leanness, LOW, the fourth Open risk restates D10: fixed as proposed: cut.
- leanness, LOW, D13's CLAUDE.md clause: fixed as proposed: cut.
- leanness, LOW, the deliverable-only acceptance criterion: fixed as proposed: replaced by the scan criterion for the PATTERN entry.
- pattern-fit, MEDIUM, batten status read bypasses PATTERN-shed-run-directory: fixed as proposed: D5 reads through the owner as a told seam.
- pattern-fit, MEDIUM, cwd-to-hub resolution against PATTERN-cwd-resolution and told-geometry: fixed as proposed: D4 makes discovery one CLI-built seam.
- pattern-fit, MEDIUM, the waiter's loop missing from the daemon-wakeups list: fixed as proposed: Constraints adds the file to `cmd/lyx/daemonwakeups_test.go` and says the confirm poll reads no pane.
- pattern-fit, LOW, the waiter spawn must be a seam under test: fixed as proposed: Constraints.
- pattern-fit, LOW, no enforcement named for the new entry: fixed as proposed: D12 marks it `(test)` with a scan, and a criterion pins it.
- pattern-fit, LOW, the WSL way forward does not reach a hibernate: fixed as proposed: D10's refusal names hibernating by hand after `lyx pause`.
- scenarios, MEDIUM, a waiter dead after `pausing` strands every hub's pause: fixed differently: the proposed orch-start or watcher notice is not made, since it puts a second reader of the record in the orch module; D8 instead lets `cancel` in any non-terminal phase with a dead waiter resume the recorded hubs, and `status` and `lyx tender status` show the stranded entries; a criterion covers it.
- scenarios, MEDIUM, entry created before it is recorded: fixed as proposed: D6 records the hub before Pause and marks it from Pause's already-paused report.
- scenarios, MEDIUM, a Pause error on one hub: fixed as proposed: D8 fails the request and resumes; a criterion covers it.
- scenarios, MEDIUM, the hidden verb has no bound: fixed as proposed: D1 gives it a token in the record and D2's caller rule; a criterion covers it. Merges leanness LOW "D1's bound is a pointer".
- scenarios, MEDIUM, a vanished status file and a held run: fixed as proposed: D5 reads a gone status file as landed, refuses a triggered request under a live hub pause, and has `status` name a held run.
- scenarios, LOW, cancel in `hibernating` and operator resume in `pausing`: fixed as proposed: D8.
- scenarios, LOW, check and write under one lock: fixed as proposed: D3.
- scenarios, LOW, a short sleep reads as a failure: fixed as proposed: an Open risk, and D8's reason says "a minute or more".
- scenarios, LOW, a hub started after discovery: fixed as proposed: D4 re-runs discovery at each confirmation poll.
- scenarios, LOW, criteria gaps: fixed as proposed: criteria for dead-waiter cancel, a held land lock, and a network operation blocking confirmation.

## Rejected alternatives

- **Batten's Land row fires the hibernate.** Rejected: one hub's batten run cannot own a pause across hubs, sits inside the process tree being paused, and the operator's direction says Go code fires after a confirmed pause on every instance.
- **The orch watcher runs the waiter.** Rejected: it couples hibernate to orch liveness and the watcher's cycle machine; a separate waiter keeps the watcher a pane writer only.
- **A machine-level hub registry written by reed or `orch start`.** Rejected: needs a new writer and goes stale; the process scan finds exactly the hubs with something to pause.
- **Scope only the requesting hub.** Rejected: the direction asks for a pause on every instance.
- **Count background-only sessions as idle in the confirmation.** Rejected here: that is `pause-background-work`'s scope.
- **Leave the hubs paused after wake.** Rejected: the operator already hibernates with runs live and expects them to carry on.
- **A halted or failed batten run meets the trigger.** First picked, then rejected at the lens review: the board's trigger is a landing, and a halted run can still be resumed by the orch, the operator or `shed-tackler`'s planned tackler.
- **The orch watcher or `lyx orch start` notifies about a dead waiter's stranded pauses.** Rejected: it makes the orch module a second reader of the hibernate record; the stranded entries already show in `lyx tender status` and `lyx hibernate status`.
- **Probe the land lock at confirmation only.** Rejected: a hub pause during a Land halts the landing, so the lock is taken before the pause and held.
- **Batten itself running `systemctl hibernate`, or any agent running it.** Rejected by the operator's direction: a session doing it is left mid-call.

## Review rounds

_No rounds yet._

## Question ledger

- Q1 What "every instance" means. Auto-pick: every lyx hub on the machine with a live lyx process, plus the requesting hub. Rationale: the direction's wording; `lyx pause` is already hub-wide.
- Q2 How to find hubs. Auto-pick: process scan through `internal/proc`, resolving each lyx process's cwd. Rationale: no registry to maintain; Linux only.
- Q3 Verb surface. Auto-pick: `lyx hibernate request [--after <slug> | --all]`, `cancel`, `status`, and a hidden waiter verb. Rationale: covers the entry's triggers and the plain verb.
- Q4 Who fires. Auto-pick: a detached Go waiter. Rationale: see Rejected alternatives.
- Q5 When the trigger is met. Auto-pick: the batten run(s) landed, reaching the terminal; a halt or failure keeps the request waiting and queues an orch notice. Rationale: the board's trigger is a landing; revised from "stopped stepping" at the lens review.
- Q6 Confirmation condition and bound. Auto-pick: every session not busy and no network operation, on every hub at one poll; ten minutes; each hub's land lock is taken before the pause and held. Rationale: the board's precondition plus the Land-in-flight point, revised at the lens review.
- Q7 Background-busy sessions. Auto-pick: count as busy; left to `pause-background-work`. Rationale: avoids duplicating that entry.
- Q8 Pre-paused hubs. Auto-pick: no new entry, never lifted by the waiter. Rationale: the waiter must not undo the operator's own pause.
- Q9 Failure handling. Auto-pick: resume own entries, mark failed, queue one orch notice. Rationale: leave hubs as they were; the operator is likely away.
- Q10 After wake. Auto-pick: detect the sleep with `SleepWatch`, resume own entries, mark done. Rationale: the operator's habit.
- Q11 Operator resume during confirmation. Auto-pick: cancels the request. Rationale: the operator's resume wins.
- Q12 Concurrent requests. Auto-pick: one per machine; refuse a second, naming cancel; re-spawn a dead `waiting` waiter. Rationale: one machine sleeps once.
- Q13 Platform. Auto-pick: native Linux only; refuse other GOOS and WSL. Rationale: the operator's platform; WSL cannot hibernate the host.
- Q14 Callers. Auto-pick: orch and external may request and cancel; agents refused, told to ask the parent. Rationale: tender's precedent.
- Q15 Shortcuts. Auto-pick: add `p` and `r`. Rationale: the entry's proposal, consistent with `a`.
- Q16 PATTERN. Auto-pick: new `PATTERN-hibernate-single-site`. Rationale: a cross-cutting invariant lands with its code.
- Q17 Record location. Auto-pick: user state directory, beside standalone state. Rationale: the request spans hubs.
- Q18 No-sleep bound after a zero `systemctl` exit. Auto-pick: two minutes. Rationale: `systemctl hibernate` returns once the job is queued; a sleep not seen by then did not happen.
