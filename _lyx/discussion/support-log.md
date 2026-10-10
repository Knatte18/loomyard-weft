# Support log: daemon-wakeups

## Interview

The session ran autonomously; no operator answered.
Each turn below is a question the design had to settle, with the evidence read and the pick made.

1. **Is this the right thing to build, or three unrelated fixes?**
   The board merges three notes under one goal, cheap waits and timely wake-ups, and labels them reed, orch and shuttle.
   Read: the three loops' code.
   Pick: one task, because Parts 1 and 2 both change how a daemon decides when to act (an event or a state, not a tick or a pane), and Part 3 is a small record change in the same packages.
2. **What counts as a daemon for the rule?**
   Read: every `time.NewTicker` and `time.Sleep` in production code.
   Found: two detached daemons (`reed watchdog`, `orch watch`), three in-run unbounded waits (batten Run-Shed, shuttle `Wait`, webster poll), one operator watch (`shed status --watch`), and many bounded waits with sub-second delays (startup handshakes, the send settle, lock retries).
   Pick: the rule covers unbounded waits and leaves bounded ones alone (D1).
3. **Where does the watchdog's idle CPU go?**
   Read: the resize loop (100 ms stat tick per worktree), the discovery loop (one `list-sessions` per hub every 5 s), the name-repair loop (two tmux calls per worktree every 10 s).
   Pick: convert the stat tick to `fswatch` (D4) and back off both spawning loops (D5), since a process spawn costs orders of magnitude more than a stat; D6 measures which dominated.
4. **How does tmux state become an event?**
   Options: tmux `session-created`/`session-closed` hooks touching a file; a control-mode client; a lyx-owned signal.
   First pick: the lyx-owned discover signal touched by the lock-held watchdog child that `up`, `resume` and `attach` spawn.
   Revised after burler round 1: other lyx paths boot sessions through `reedengine` without spawning the watchdog, so the signal is touched by `reedengine`'s cold session boot instead (D5); tmux session hooks and control mode rejected below.
5. **What held the reload on 2026-10-10?**
   Read: the prime's `.lyx/orch/watch.log` (read-only) and the watcher.
   Found: four `session state disagrees with the idle probe` lines with `idle=false state=asking`, the first one second after the boundary, and no reason for the pane reading; every injection gate in the watcher is `probeIdle`, which is the pane capture alone; the state is logged, never acted on.
   Pick: the gate moves to a hook-derived readiness reading with one pane check for a draft (D7), and the reason is logged (D8).
6. **Can the pane be dropped entirely?**
   Read: `IdleSession` (needle, box found, box blank) and `InputBoxText`.
   Pick: keep the draft check, because a draft is the one hazard hooks cannot see and typing over the operator's text is worse than a late reload; drop the needle and box-found holds when hooks say the turn ended.
7. **Should the verified send change too?**
   Read: `awaitIdleSession` and `busyReading` poll the same pane probe up to 60 s.
   Pick: yes, the same reading in both, else the watcher would inject and the send would then refuse for the same pane reading (D7).
8. **Queue the resume steps ahead of the operator's message?**
   First pick: no; with the gate fixed the steps land within one 2 s poll of the boundary, and queued slash commands in Claude Code's input are unverified.
   A draft holds, with the reason logged.
   Revised after parent review round 1: the pointer rides Claude Code's session-start hook on the compaction matcher, so it is in the context ahead of the operator's message whatever the box holds (D11); typing into the box while a turn runs stays rejected.
9. **Should the orch watcher itself move to events?**
   Pick: no; its 2 s poll complies with the floor and its idle tick reads files only.
   The lateness was the gate, not the cadence.
10. **Floor or refuse a sub-second config interval?**
    Pick: floor with one Warn (D9), the degrading side of config strictness; shuttle's template default moves to 1000 ms so the shipped values comply.
11. **Where does the start time live?**
    Options: `seed.json`, reed's strand record, a new file.
    Pick: `seed.json`, the run record spawn order already reads, stamped by `shedrun.WriteSeed` so no seeding site changes (D10); no mtime fallback, since the mtime is the fault.
12. **How is the guard shaped?**
    Pick: a declared-file literal scan plus a ticker scan over every production file, with `//lyx:one-shot` as the announcement, and per-loop floor tests for the computed cases (D2).

## Rejected alternatives

- **tmux `session-created`/`session-closed` hooks** writing a hub signal for discovery.
  Rejected: server-level hooks must be installed and verified on every server start, the Windows `psmux` path cannot run them, and the lyx-owned discover signal covers every session lyx boots with no new mechanism.
- **A tmux control-mode client** per hub for session and pane state.
  Rejected: a long-lived tmux client process per daemon, a parser for the control protocol, and no Windows path; far more than the measured cost warrants.
- **Watching the Claude transcript with `fswatch`** so the orch reload fires within milliseconds of the boundary.
  Rejected: the transcript lives outside lyx dirs, the 2 s poll complies with the rule, and the delay in #481 was minutes of a blocked gate, not seconds of a poll.
- **Typing the resume steps into Claude Code's input box** during the operator's turn so they queue.
  Rejected: whether a queued slash command runs as the next input is unverified, and typing over a draft is the worse failure; the session-start hook (D11) gets the pointer ahead of the message without touching the box.
- **Delivering `/color` and `/reload-plugins` through the hook too.**
  Rejected: they are commands the session must execute, which a hook's context cannot run; they stay typed and readiness-gated, and their lateness under a draft costs stale plugins for one turn, not a session without its role.
- **Dropping the typed pointer step once the hook exists.**
  Rejected: the hook's firing and ordering are assumed from Claude Code's documented behaviour, not proven in this repo; the typed step stays as the fallback until a `session-start` event confirms delivery.
- **Dropping the pane entirely from the readiness reading.**
  Rejected: a draft is invisible to hooks and typing over the operator's text is the worse failure.
- **Keeping the pane probe and fixing its recognition** of the post-compaction screen.
  Rejected: the cause of the 2026-10-10 reading is not recorded, so a recognition fix is a guess; moving to the hook-derived state is the direction `shuttle-state` already decided.
- **Refusing a sub-second configured interval** at config load.
  Rejected: a refusal stops a daemon over one key; floored with a Warn instead.
- **A runtime wake-up counter test** driving each loop for simulated minutes of idleness.
  Rejected as the primary guard: it needs every loop to take an injectable clock and event source before it can be written, which the two unconverted loops lack today; the floor tests cover the computed cases.
- **Falling back to the seed's mtime** when `started_at` is absent.
  Rejected: the mtime is what moves; undated-by-name is explicit and the cost is one restart's ordering for runs seeded before the change.
- **A hub-level name-repair pass** using one `list-panes -a` for every worktree.
  Not rejected, left to the plan writer as an option: it would cut spawns further but reshapes the per-engine op lock; D5's backoff alone meets the rule.
- **Converting batten's Run-Shed wait, shuttle's `Wait` and webster's poll to `fswatch`.**
  Rejected for this task: each complies at its floor, the board names the reed watchdog first, and `parent-review-hardening` already owns the shuttle gate's poll cost.

## Review rounds

### Parent review, round 1

Verdict BLOCKING, one finding.

- **F1 (BLOCKING, scope): the resume steps can still run after the operator's next message when a draft is in the box.**
  Fixed.
  D11 added: the resume pointer is delivered through the provider's session-start hook on the compaction matcher, rendered by a new `lyx orch resume-context` verb and recorded in the events file, so it is in the context ahead of the operator's next message; the typed pointer step stays as the fallback when no `session-start` event is read.
  D7's Bound now says only `/color` and `/reload-plugins` wait on a draft.
  The Scope "Out" bullet on queuing is replaced by two narrower exclusions, typing into a running turn's box and using the hook after `/clear`.
  Acceptance criteria added for the draft case, the hook fallback, the verb and the hook rendering; an assumption and an open risk name the hook semantics the design rests on.

### Burler round 1

Verdict BLOCKING, ten findings, all fixed.

- **F1 (BLOCKING, design): the skip keyed on the recording command's event, so a failed render left no pointer.** D11 now skips only on a delivery mark the verb writes after a successful render and whose text equals `PendingResume`; the double delivery a late or differing mark causes is stated as harmless.
- **F2 (BLOCKING, design): the phase alone did not settle resume versus reload.** D11 renders `PendingResume` in `resuming`, the resume prompt in `compacting`, the reload prompt otherwise, and states what a compaction in `handoff-requested` or `clearing` gets.
- **F3 (BLOCKING, design): not every lyx boot spawns the watchdog.** D5 moves the touch to `reedengine`'s cold session boot; the Interview turn, Q3 and the acceptance criterion follow, and the assumption is dropped.
- **F4 (MEDIUM, design): the discover signal had no bound.** D5 gains a Bound: wakes run no faster than the 5 s base, so the worst case is today's cost.
- **F5 (MEDIUM, design): no release for an unmatched turn start.** D7 adopts `turnStartHold`'s releases, bounds the idle override, and a criterion covers it.
- **F6 (MEDIUM, consistency): D7's lists missed `unknown` and `dead`, and keyed on the newest signal.** D7 reads the folded state, covers every state name, and says a `session-start` signal leaves the reading unchanged; the criterion mirrors it.
- **F7 (LOW, consistency): consuming the signal could wake the loop twice.** D5 stats first, and an event that finds no file runs nothing.
- **F8 (LOW, consistency): the note and D7 disagreed on `IdleProbe`.** D7 keeps `IdleProbe` with a new `Reason`; the note agrees.
- **F9 (LOW, consistency): the fsnotify pin lived only in an assumption.** Moved into D3.
- **F10 (NIT, consistency): Open risks restated Decisions.** Cut to the residual risks: watching the first smoke run and the first live compaction.

### Burler round 2

Verdict APPROVED, four findings, all fixed.

- **F1 (MEDIUM, design): the verb resolved the prime from the hook's cwd, which drifts into other worktrees.** D11's hook command now changes to the prime's absolute path, built through `internal/shell`, before running the verb; the assumption and a criterion follow, and `PATTERN-shell-mechanics-seam` joins the Constraints.
- **F2 (MEDIUM, design): under a draft the hook's pointer named a role file the reload had not re-rendered.** The verb renders the role file before the pointer in every phase but `resuming`, `RenderRoleFile` writes through a rename, and the verb's criterion covers it.
- **F3 (LOW, design): the hook's `lyx` binary was unstated.** D11 states that the bare `lyx` resolves to the spawning binary through the pane prelude's `PATH` entry and `LYX_BIN`, which the hook inherits.
- **F4 (LOW, design): D4's event wake had no bound.** D4 gains a Bound (writer, per-event cost, no-file wake, apply rate), and the resize-loop criterion covers the no-file wake.

## Question ledger

- **Q1 (auto-pick): scope of the rule.** Unbounded waits only; bounded waits keep sub-second delays. Rationale: the measured cost is hours of nothing happening; bounded waits are correct at sub-second cadence.
- **Q2 (auto-pick): the file-event dependency.** `github.com/fsnotify/fsnotify` v1.9.0, wrapped in a leaf package. Rationale: the board asks for one shared cross-platform package; fsnotify is the standard and is in the module cache.
- **Q3 (auto-pick, revised after burler round 1): discovery event source.** A lyx-owned discover signal touched by `reedengine`'s cold session boot, not tmux hooks and not the lock-held spawn child. Rationale: no server-side install, works on Windows, and the boot is the one path every lyx-booted session takes, where the spawn child misses boots that never spawn the watchdog.
- **Q4 (auto-pick): backoff shape.** Doubling from the base to a 60 s ceiling while nothing changes, reset on change or event. Rationale: the rule's clause for spawning wakes; the reap's three-cycle count tolerates a slower cadence.
- **Q5 (auto-pick): cause of the 2026-10-10 pane reading.** Unknown; the prime's log records no reason. The design holds under a probe misread and under a draft, and D8 records the reason next time.
- **Q6 (auto-pick): `busy`/`background` reads ready.** Rationale: Claude Code takes a prompt while background shells run; the orch's notices and reloads should not wait on them.
- **Q7 (auto-pick): `asking`/`ask` holds.** Rationale: a dialog is open and typed text would answer it.
- **Q8 (auto-pick): shuttle's wait default to 1000 ms.** Rationale: the shipped value must comply with the floor the guard pins; one poll of added latency at most.
- **Q9 (auto-pick): `started_at` stamped in `WriteSeed`.** Rationale: one site, every caller covered, the codec's sole declarer.
- **Q10 (auto-pick): no mtime fallback for undated seeds.** Rationale: the mtime is the fault; one restart's ordering for pre-change runs is the only cost.
- **Q11 (auto-pick): the measurement procedure.** `/proc/<pid>/stat` over ten minutes, shell only, report under `docs/benchmarks/`. Rationale: lyx agents never run Python; a measurement report may pin numbers.
- **Q12 (auto-pick): the orch watcher stays a poll.** Rationale: 2 s complies; the idle tick reads files only.
- **Q13 (auto-pick, after review round 1): the pointer's hook route.** `SessionStart` with matcher `compact`, two commands (the recording hook and `lyx orch resume-context`), the typed pointer as fallback. Rationale: the reviewer's suggested route meets the board's first expectation in the draft case; the recording command gives the watcher a confirmation and the fallback covers a hook that does not fire.
- **Q14 (auto-pick): the hook covers compaction only, not `/clear`.** Rationale: the clear cycle needs a typed skills load anyway, and the board's expectation is about compaction.
- **Q15 (auto-pick, burler round 1): what confirms the hook delivered the pointer.** A delivery mark the verb writes after a successful render, matched against `PendingResume`, not the recording command's `session-start` event. Rationale: the event is written whatever the verb does, so keying on it made the fallback unreachable on a failed render.
- **Q16 (auto-pick, burler round 1): what the verb renders.** `PendingResume` in `resuming`, the resume prompt in `compacting`, the reload prompt otherwise. Rationale: the watcher can enter `resuming` before the hook runs, and the phase alone then loses the handoff note.
- **Q17 (auto-pick, burler round 1): releasing an unmatched turn start in the watcher.** `turnStartHold`'s releases, unchanged. Rationale: turn starts without a turn end are routine, and one release rule for the watcher and the send keeps them from disagreeing.
- **Q18 (auto-pick, burler round 2): how the hook reaches the prime.** A change of directory to the prime's absolute path in the command, not a new directory flag. Rationale: no orch verb takes a directory flag, and resolving through `lyxcwd` after the change keeps the verb resolving as every orch verb does.
- **Q19 (auto-pick, burler round 2): who renders the role file the hook's pointer names.** The verb, in every phase but `resuming`. Rationale: under a draft the watcher's render waits on readiness, and the skip then leaves the session on the previous role.
- **Environment:** `lyx board list` and several file reads exceeded the tool's output size and were read from the persisted output files; nothing was broken and nothing was repaired.
