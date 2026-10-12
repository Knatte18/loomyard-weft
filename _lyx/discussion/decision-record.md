# Decision record — lyx-halt

## Goal

Give the operator one ordered, intentional pause that makes the hub safe to suspend within seconds.
`lyx pause` interrupts every running Claude session with Esc after a short grace, holds every path that would start agent or network work, and is recorded so that no failure, idle or crash path fires for it.
`lyx resume` lifts it and lets each interrupted session take up its turn where it stood.
The bare loom and webster boundary pause goes, and an unplanned sleep stops counting as idle time or killing work on wake.

## Scope

In:

- A new module with `lyx pause [<slug>]` and `lyx resume [<slug>]`, hub-wide without a slug and per run with one, and `lyx pause --status`.
- The pause record, and the read of it by every path that reacts to an unexpected stop or to idle time.
- Holding every agent-session spawn path, typed resumes of a parked driver, and network git and GitHub calls while a pause covers them.
- Retiring the bare boundary pause of `lyx loom pause` and of `lyx shed pause` against a loom run, and `lyx webster pause` with webster's pause flag; `lyx batten pause` stays.
- The orch session, interrupted and resumed through the orch watcher.
- Unplanned sleep: one shared sleep detection, awake-time idle in the shed loop watchdog and shuttle's deadline, a once-per-turn automatic continue for an API-error stall across a detected sleep, and burler's live-fix resume checking that the task prompt was delivered.
- Issue #567: a watchdog kill names itself in the halt note and trace, and the killed step's shuttle runs get a terminal record.

Out:

- Killing or suspending Go processes: an in-flight Go producer runs on locally under a pause, and a run advances through local Go rows until it reaches a held path.
- A systemd-logind sleep hook or any automatic pause before a suspend.
- Creel: this task types through the existing shuttle and orch-watcher paths.
- Claude sessions that shuttle did not start, such as a hand-added `lyx reed add --cmd claude` strand.
- Claude Code's own `SendMessage` into a covered session, which lyx cannot intercept.

## Decisions

### D1 — One module, two root commands

A new CLI module `pause` owns `lyx pause [<slug>]`; `lyx resume [<slug>]` is mounted in `cmd/lyx/main.go` as an alias command delegating into the module's resume command, the alias form `PATTERN-cli-cobra` admits.
Both are operator-audience, run from any worktree of the hub, and resolve the hub through `internal/cliwire`.
`lyx pause --status` prints the live entries and changes nothing.
`PATTERN-shed-verb-set` does not apply: these are hub verbs, not shed verbs.
The orch and an `EXTERNAL` caller may run them with or without a slug.
An agent, a caller whose `LYX_STRAND_NAME` names a shuttle-run strand of a pair, may run `lyx pause <slug>` and `lyx resume <slug>` only for its own pair's slug; a hub-wide `lyx pause` or `lyx resume` from it, or either naming another pair's slug, refuses, naming the way forward of asking its parent session, which runs `lyx pause` or `lyx resume` for it.
Caller identity is read from the strand-name environment variable, and the agents' settings deny any command naming it, so an agent cannot clear it to read as `EXTERNAL`; the check guards against a mistaken call, not a determined agent.
Input:

- A slug with no pair in the hub refuses, listing the slugs that have one; a slug whose pair has no running run is accepted and covers whatever starts there.
- `--grace` takes 0 to 60 seconds, default 10; a value outside that range refuses.
- `lyx resume <slug>` with no live entry for the slug and `lyx resume` with none at all are no-op successes reporting nothing resumed.

Rationale: the operator named `lyx pause`; a hub-wide verb belongs to no existing module, and `loom resume` and `reed resume` keep their run-scoped meaning.

### D2 — The pause record and where it lives

The record is one JSON file at a path `internal/lyxdirs` declares under the hub's board-level `.lyx`, written under a lock beside it.
It holds live entries and resumed entries.
Each entry has a scope (`hub` or a slug), `paused_at`, the caller's strand name or `EXTERNAL`, and the sessions it owns (strand name, guid, pair slug or the orch).
A resumed entry keeps its interval with `resumed_at` for 7 days, so a clock can subtract it after the resume; an idle clock reaching back further than 7 days counts that pause as idle time.
A run is covered while a live entry has scope `hub` or its slug.

Two packages:

- `pauserecord`, a leaf: the record's read, write and coverage query, the paused intervals, and the shared sleep detection (D9). It imports the standard library and `lock` only, and `PATTERN-leaf-packages` gains its row.
- `pauseengine`, the module engine under `pausecli`: the pause and resume sequences of D3 and D6, over shuttle and reed.

The record's path enters the hub and standalone geometries (`hubgeom`, `standalonegeom`) and is handed to every reader as a told path, so no bound engine derives it (`PATTERN-told-geometry`).
Rationale: one durable fact read by many modules; it survives a reboot, so a suspend that ends in a reboot stays paused until `lyx resume`.

### D3 — The pause sequence

`lyx pause` runs, in order:

1. Write the entry, with its grace end. From here every reader in D5 treats the covered runs as ordered-paused.
2. Grace: wait until every in-flight network operation (D4) has finished and every covered agent session reads idle, or until the grace has passed.
3. Interrupt, per covered session that reads busy: under the record lock, check that the entry is still live and that no live entry already owns the session, record the session in the entry, release the lock, then play shuttle's interrupt choreography (`Runner.Interrupt`). Read the state once more and interrupt once more if still busy.
4. Report: sessions interrupted, already idle, still busy after the retry, and refused by `Runner.Interrupt`'s pane check; network operations still in flight, naming those under a parent lock (D4); each parent branch a covered run holds and the runs queued behind it (D5); each covered run waiting in a parent lock's queue and the runs queued behind it (D4); covered runs whose current producer is a Go producer; and, under a hub entry, the orch's case: delegated to a live orch watcher, or not interrupted because no watcher is live (D7).

A resume that removes the entry during steps 2 or 3 stops step 3 at the next session: no Esc goes after the entry is gone, and the report says the pause was resumed during its grace.
A session is recorded before its Esc, so a pause killed between the two leaves a recorded session that may not have been interrupted; D6 bounds what that costs.
Covered sessions are the live shuttle-run strands of every covered pair, the loom driver included; under a hub entry the orch is covered through D7 and never touched by this verb.
The caller's own strand (`LYX_STRAND_NAME`) is never interrupted, since an Esc would kill the verb's Bash call; the report names it as skipped.
A second `lyx pause` with the same scope while one is live adds no entry and runs steps 2 to 4 for sessions busy now, recording them in the existing entry.
A `lyx pause <slug>` under a live hub entry adds the slug entry and owns only sessions no live entry owns yet.
Bound: the verb sends Esc only, never text, only to sessions shuttle tracks and no live entry already owns; it kills no pane, strand or process.
Rationale: the record goes first so no Esc reads as a failure, and the grace lets a running Bash command or push end cleanly.

### D4 — What a pause holds

While a live entry covers a run, each path below waits or refuses:

| Path | Under a covered run |
|---|---|
| Shuttle run start (every producer, advisor, Burler round and burler relaunch goes through it) | waits at entry until no entry covers the run; its deadline starts when it starts |
| `lyx reed resume` and batten's revive of a dead driver through reed resume | skip the covered pair's strands, leave them not-live and report them; they are replayed after the resume as today |
| `lyx loom start`, `lyx batten run` and `lyx loom resume` on a covered slug | refuse, naming `lyx resume <slug>` or `lyx resume` |
| `lyx shuttle send` into a covered strand | refuses, naming `lyx resume` |
| Network git in `internal/fabricengine` (push, fetch, pull, a landing's remote calls) and every `internal/githubclient` call | wait at entry |
| `lyx orch start` | not held: it is the operator's own session, and an orch started under a hub entry is not interrupted |

Reed's server revive relaunches no strand, so it needs no hold.
A hub entry holds every network operation; a per-run entry holds those run on behalf of that pair, except one whose caller holds a parent lock (`internal/parentlock`).
A covered run is never granted the parent lock while covered: its lock wait keeps waiting until the resume, and the pause alone never ends it with an `ErrPaused` halt (a `--after` naming the waiting row still does, D8), so the exception reaches only a run that already held the lock when the entry was written.
A covered waiter yields its turn: it keeps its ticket and its place in the queue, but a free record goes to the oldest live ticket whose run no entry covers, so an uncovered run queued behind a covered waiter is granted as if the covered one were absent; each ticket names its run's slug so the grant can read coverage, and a covered waiter still first in the queue at the resume is granted next.
It lets through that holding run's remote calls of its current landing row, and D3 step 4 and `lyx pause --status` name each as a network operation in flight under a parent lock.
The exception keeps a held network call of the holder's current row from extending its hold on the lock; other runs still wait behind a covered holder until the resume (D5).
Each network operation in flight has a marker under the record's directory that it removes on finish, so D3's grace can wait for it; a marker whose pid is dead is ignored and removed.
Every wait wakes under `PATTERN-daemon-wakeups` and has no deadline of its own.
A call held at entry prints one line to its stderr, or to its log when it has no terminal, naming the pause it waits on and `lyx resume` as the way forward, once when its wait starts, so an operator's own `lyx board` write or `lyx selfreport create` under a hub entry does not hang silently.
Bound: these are waits at entry points and refusals that name `lyx resume`; none fails work, the worst case is a run waiting until `lyx resume`, and `lyx pause --status` names what waits.
A `SendMessage` into a covered session is not held; lyx's own senders do not send while paused (D5, D7).
Rationale: local Go work survives a suspend, while a network call cut by one hung batten for 2.5 hours (#458) and an agent turn is what an API error ends.

### D5 — Ordered, not failed: the readers of the record

Each reader below treats a covered interval as neither idle time nor a failure:

- Batten's InnerRun notices: no quiet, api-error or driver-dead notice for a covered run, and the idle clocks subtract every paused interval. A state-changed notice still goes, since a halt during a pause is a real halt of a Go producer.
- Shuttle's Wait: no hold notice and no dialog notice for a covered run, and the deadline excludes paused intervals (and slept time, D9).
- The shed loop's step watchdog: no kill while covered, and paused time is not idle time.
- The parent lock's idle release: paused time is not idle time for a covered holder, so the lock stays held until the resume, and other runs' lock-taking rows wait in the queue as for any live holder. `lyx pause --status` and D3 step 4 name each parent branch a covered run holds and the runs queued behind it, and each covered run waiting in a parent lock's queue and the runs queued behind it, which D4's yielded turn lets past.
- The orch watcher: under a live hub entry it holds its notice queue and starts no context cycle, and delivers the held notices after the resume.
- Burler and seat rounds and the loom driver's interrupted-stop diagnosis need no change of their own: their agents are shuttle runs, so they are held by Wait, and no step is interrupted, so no interrupted stop is written.

No status file changes state for a pause: a covered run still reads `running` at its current producer.
`lyx loom status` and `lyx batten status` carry `paused_since` from the record for a covered run.
Rationale: the pause freezes runs rather than stopping them, so no producer is re-invoked and loom's `interrupt_policy` is not consulted.

### D6 — Resume

`lyx resume` moves the covering entry to the resumed list under the record lock, then, for each session the entry owns:

- When another live entry covers it, ownership moves to that entry and no line goes.
- When its strand is gone or dead, no line goes and the envelope names it; reed resume and batten's driver-dead path take it after the resume as usual.
- When it reads busy (the operator typed into it), no line goes and the envelope names it.
- Otherwise one fixed resume line goes through shuttle's guarded paste-submit (`PATTERN-pane-text-injection`), such as `[lyx] A pause interrupted your last turn and is now over. Carry on where you stopped: rerun the tool call or fork the pause cut. If nothing was cut, end your turn.`
- When the guarded paste-submit refuses (a draft in the box), no line goes and the envelope names the session with the reason.

`lyx resume <slug>` resumes that slug's entry; when only a hub entry covers the slug it refuses, naming `lyx resume`.
`lyx resume` with no slug resumes every live entry.
Bound: at most one line per session per resume, only into a session the entry owns; a session recorded by a pause killed before its Esc gets one surplus line, which its text tells it to end on.
Rationale: an interrupted Claude turn ends at the prompt, and in-process forks and the tool call in flight end with it, so the line names them; a session idle at pause time was waiting on nothing.

### D7 — The orch session

Under a hub entry the orch is covered, and only the orch watcher touches it (`PATTERN-orch-pane-single-writer`).
On its existing tick the watcher reads the record.
On a new live hub entry it ends the orch's grace by the rule D3 step 2 states: early once the orch reads idle and no network operation marker of the orch's is open, which leaves nothing to interrupt, and otherwise at the grace end D3 step 1 records in the entry.
At that grace end it plays the interrupt choreography into a busy orch, at most twice, after checking the entry is live and recording the orch in it, the order D3 uses.
On that entry's resume, when the entry owns the orch, it queues the resume line as a notice, delivered idle-gated; an orch the entry does not own, idle at pause time or the caller, gets none.
When the entry's caller is the orch, the watcher does not interrupt it.
`PATTERN-orch-pane-single-writer` changes in the same commit to: only the orch watcher types into the orch session, idle-gated, except the interrupt a hub pause asks for.
Rationale: the orch makes network calls and runs landings, so a suspend-safe hub includes it, and routing it through the watcher keeps one writer.

### D8 — Which boundary pause goes

- `lyx loom pause` with no flag refuses, naming `lyx pause <slug>` to stop the agents and `lyx loom pause --after <current_producer>` to halt at the next boundary; `lyx shed pause` against a loom run does the same. `--before`, `--after` and `--clear` stay.
- `lyx webster pause` and webster's pause flag go, with the refusal-spec row "Webster row paused out of band".
- `lyx batten pause` stays as it is, `pause_requested` stays in `shedengine` for it, and batten's `PauseRequested` seam stays: it ends batten's own wait within one check, mid-row, which the orch's hold-then-goto (`lyx batten pause <slug>`, then `lyx batten goto`) needs and which no flag form gives.
- landingshed's `PauseRequested` seam reads the run's `pause_after` naming the waiting row instead of `pause_requested`, so `lyx loom pause --after <row>` ends a lock wait with the existing halt; `--after` thus no longer means only a halt after the named row finishes: naming a row waiting for the parent lock, it ends that wait mid-row; the refusal-spec row "parent lock wait paused" stays, its trigger reworded to that condition.
- The loom ways forward that named the bare pause (shed busy, goto on a running run) name `pause --after <current_producer>` for a loom run; for a batten run they keep `lyx batten pause`. The arbiter-attempt refusal drops its "a requested pause" clause for loom.
- The orch stencil's "pause both after Plan-Write (`lyx loom pause` while Plan-Write runs)" becomes `lyx loom pause --after Plan-Write`, a boundary halt a goto can move.

Bound: the only mid-row halt left for a loom run is a parent lock wait ended by `--after` naming the waiting row; the agents of a loom run are stopped mid-turn by `lyx pause <slug>`, and a loom run halts for a goto at its next boundary through `--after`.
Rationale: the operator's complaint was that the boundary pause lets an agent turn run on; that is loom's and webster's pause, whose use `lyx pause` and `--after` now cover, while batten's pause stops a Go wait, not agent work, and has a use nothing else covers.

### D9 — Unplanned sleep

- Batten's awake-versus-wall sleep detection moves to `pauserecord`, and batten, the shed loop watchdog and shuttle's Wait read slept time through it.
- The shed loop watchdog measures idle in awake time: its idle reading today is the wall-clock time since the newest trace-file mtime or agent activity, so a wake after a sleep longer than its window kills at once.
- Shuttle's deadline is measured in awake time, so a night's sleep does not time a run out on wake.
- Batten's quiet and api-error notices after a detected sleep count awake time only; the plan checks why the 2026-10-12 notice counted almost eleven hours despite the existing subtraction, and fixes the cause.
- Shuttle's Wait: when the newest turn end is an API-error stall and a sleep of at least one minute was detected during that turn, it types one fixed continue line through shuttle's guarded paste-submit (`PATTERN-pane-text-injection`), once per turn, and when the guard refuses the write nothing goes and the stall is held and notified as today; a second API-error stall on the same turn is held and notified as today. It never fires for a covered run.
- Burler's live-fix resume (`burler: resuming a live fix`) attaches to a live strand only after reading that the task prompt's turn started in it; when only the skill-load preamble did, it delivers the task prompt first.

Bound: the automatic continue fires only on an API-error cause, only across a detected sleep, at most once per turn, and only through the guarded paste-submit, sending nothing when it refuses.
Rationale: the board's seen case is an API error across a sleep, a watchdog kill on wake and a fix chair left without its task.

### D10 — Issue #567: a watchdog kill names itself

- The shed loop's stop for a watchdog kill carries the watchdog (the step watchdog), its idle window, the last activity time and the idle time measured, in the halt note's reason and the step's trace.
- The kill ends the step's process group, but its agents live in reed panes outside it, so their `run.json` stays `running`. After a watchdog kill the arming module (`loomcli`'s loop arming, not the generic `shedverbs`) stops every live shuttle run of the killed step through shuttle's stop verb (`PATTERN-shuttle-stop`), which records `stopped` and removes the strand.
- So the step's next relaunch starts its agent fresh; D9's live-fix reattach applies only to a live strand left by a death other than a watchdog kill.

The shuttle run record (`run.json`) gains a field naming the shed run directory and step that started it, written by shuttle's Start from what the step hands its producer, empty for a run no step started, and read by the arming module to find the killed step's runs.
Bound: the stop takes only shuttle runs whose record names the killed step, never the loom driver's strand, the orch or a run of another step; the driver's strand and the orch are started by no step, so their records carry no step, and another step's runs name that step.
A live run whose record names no step, such as one an older binary wrote, is left and named in the halt note.
Rationale: the issue asks for these, and a kill that leaves a live agent behind is what made the relaunch reattach to a strand without its task.

## Constraints

- `PATTERN-no-denied-recovery`: no way forward names a command the agents' settings deny; the plan checks `lyx resume`, `lyx pause` and `lyx loom pause --after` against them.
- `PATTERN-refusal-way-forward`: every new refusal (bare loom pause, `lyx shuttle send`, `lyx loom start`, `lyx batten run` and `lyx loom resume` under a pause, per-run resume under a hub entry, a bad slug, a bad grace, a hub-wide pause or resume or another pair's slug from an agent, whose way forward is asking its parent session) lands its row in `contracts/specs/refusal-spec.md` with a test, and the rows D8 changes or removes are rewritten in the same commit.
- `PATTERN-fabric-git` and `PATTERN-github-auth`: the network holds sit inside `fabricengine` and `githubclient`, not at their callers.
- `PATTERN-daemon-wakeups` for every wait the pause adds.
- `PATTERN-sandbox-coverage`: the new module is exercised by the sandbox suite or excluded with a reason.
- Same-commit edits beside the code:
  - `pauseengine`'s and `pauserecord`'s `doc.go`, and `docs/overview.md`'s module table;
  - the batten, shed, shuttle, reed, orch and webster package docs that describe the boundary pause or the paths D4 and D5 change;
  - `contracts/stencils/orch/orch-template-role.md` (line 90's loom pause) and the shed driver stencil `contracts/stencils/shed/shed-template-driver-guide.md`, whose "never `lyx shed pause` as a repair" stays true;
  - `contracts/stencils/claims_test.go` claims that pin those lines;
  - `contracts/specs/loom-status-spec.md`, `contracts/recipes/batten-recipe.yaml`, `docs/skills.md` and the sandbox suites where they name the loom or webster pause.

## Auto-mode assumptions

- The orch is covered (D7) and per-run `lyx pause <slug>` exists (D1), though the board left both open.
- `lyx batten pause` is kept (D8) against the board's "the step-boundary pause goes", read as aimed at the pause that lets agent work run on.
- 10 s of grace reads "a few seconds" with room for a push to finish.
- Leaving Go producers running reads as compatible with "safely stops every lyx process", since their local work is safe across a suspend and their network work is held.

## Open risks

- The creel task will move `lyx shuttle send`, the orch watcher's typing and `PATTERN-orch-pane-single-writer` behind creel; creel's delivery must then honour pause coverage, holding injects and reminders for a covered receiver, or a creel inject wakes a frozen agent. The creel entry does not say so yet.
- `runs-overview` (in flight) shows one line per run with its state; a covered run reads `running` with a growing age unless the overview reads `paused_since` (D5).
- A hub-wide pause the operator or the orch leaves on silences every notice; `lyx pause --status` and `paused_since` are its only signals.

<!-- lyx:carry-over Discussion-Review -->
- Discussion-Review, round 2: converged; each finding below was fixed or disputed by the round's fixer, and no fresh reviewer has seen the result; the fixer report says which.
  Review: _lyx/reviews/discussion/round-2-review.md; fixer report: _lyx/reviews/discussion/round-2-fixer-report.md.
  - agent-refusal-sends-agent-to-operator (class decision, severity MEDIUM, open in rounds 2)
  - covered-waiter-blocks-parent-lock-queue (class design, severity MEDIUM, open in rounds 2)
  - d10-shuttle-tie-names-absent-field (class design, severity MEDIUM, open in rounds 2)
  - d7-drops-grace-for-busy-orch (class consistency, severity MEDIUM, open in rounds 2)
<!-- /lyx:carry-over Discussion-Review -->

## Acceptance criteria

Pause and resume:

- `lyx pause` writes the entry before any Esc, waits at most the grace, interrupts each busy covered session at most twice, skips the caller's strand, and reports each session's case and open network operations.
- With nothing running it writes the entry and reports empty lists; a repeated pause adds no entry and interrupts only sessions busy now.
- A resume during a pause's grace stops any further Esc, and the report says so.
- A pause killed after recording a session and before its Esc leaves the session to get one resume line, which it ends on.
- Hub and slug entries overlapping: a session is interrupted once, by the first entry; resuming that entry while the other is live moves the session to it with no line, and the line goes when the last covering entry resumes; `lyx resume <slug>` under only a hub entry refuses with its way forward.
- `lyx resume` sends one line into each owned session and none into a dead, busy, idle-at-pause or draft-holding one, naming each skipped one with its reason.
- A record survives a restart and a reboot: after a reboot under a live entry, `lyx reed resume` leaves the covered pair's strands down and `lyx batten run` on it refuses.
- Concurrent pauses and resumes leave a well-formed record and no Esc after an entry's resume.
- Invalid input: an unknown slug and a grace outside 0 to 60 refuse with their ways forward; resuming nothing is a no-op success.
- Caller: a hub-wide `lyx pause` or `lyx resume` from an agent strand, and either naming another pair's slug from one, refuse with the way forward of asking its parent session; the orch's hub-wide pause succeeds, and an agent's pause of its own slug succeeds.

Holds and readers:

- Under a covered run: a shuttle run start, a fabric push or fetch and a GitHub call wait and proceed after the resume, a started run's deadline counting from its start; `lyx loom start`, `lyx batten run`, `lyx loom resume` and `lyx shuttle send` refuse; batten's driver revive skips the pair.
- Under a per-run entry, a network operation whose caller holds a parent lock is not held, and the pause report and `lyx pause --status` name it.
- A covered run waiting in the parent lock's queue is not granted the lock until the resume; under a per-run entry an uncovered run queued behind it is granted a free lock while the covered run keeps its place, and the pause report and `lyx pause --status` name the covered waiter and the runs behind it.
- A call held at entry under a hub entry, such as an operator's `lyx board` write, prints one line naming the pause and `lyx resume` when its wait starts, and no more while it waits.
- A covered run holding the parent lock keeps it through the pause with no idle release; another run's lock-taking row queues behind it until the resume, and the pause report and `lyx pause --status` name the held branch and the queued run.
- While paused: no batten quiet, api-error or driver-dead notice, no shuttle hold or dialog notice, no step watchdog kill and no orch-watcher delivery or cycle, however long the pause lasts; a state-changed notice still goes for a run that halts.
- After resume: idle clocks and the shuttle deadline exclude the paused interval, and a run idle for its window after the resume is noticed as before.
- The orch: under a hub entry a busy orch, including one running a local command with no network operation open, is interrupted by the watcher only at the entry's recorded grace end, an orch that reads idle with no network operation of its own open is not interrupted, and an interrupted orch gets the resume line as a notice; an orch that ran `lyx pause` is not interrupted; an orch idle at pause time, or that ran `lyx pause`, gets no resume line; the pause report says whether the orch was delegated to a live watcher or not interrupted because no watcher is live.

Boundary pause:

- A bare `lyx loom pause`, and a bare `lyx shed pause` against a loom run, refuse with both ways forward; `--before` and `--clear` work as before, and `--after` works as before except that, naming a row waiting for the parent lock, it ends that wait with the halt of D8; `lyx batten pause` ends batten's wait within one check as before; a status file carrying `pause_requested` still loads.
- Goto and shed-busy refusals name `pause --after <current_producer>` for a loom run and `lyx batten pause` for a batten run.
- A loom row waiting in the parent lock's queue halts on `lyx loom pause --after` naming it, and `lyx loom goto` then moves the run.

Sleep and #567:

- After a detected sleep longer than its window, the step watchdog does not kill a step whose awake idle time is under the window, and a shuttle run does not time out on the slept time.
- A batten quiet notice after a sleep counts awake time only.
- An API-error stall across a detected sleep gets one continue line; a second on the same turn is held and notified.
- Burler's live-fix resume of a strand holding only the skill-load preamble delivers the task prompt.
- A watchdog kill's halt note and trace name the watchdog, window, last activity and idle time, and the shuttle runs whose record names the killed step read `stopped`; the loom driver's strand and runs of other steps stay live.

## Notes for the plan writer

- Pieces to build on: shuttle's `Runner.Interrupt` and `StopStrand`, batten's `sleepGap` and `Awake` seam in `internal/battenshed`, the step watchdog in `internal/shedverbs/loopwatchdog.go`, the generic pause verb in `internal/shedverbs/pause.go`, webster's pause flag in `internal/websterengine/pause.go`, the orch watcher in `internal/orchengine`.
- The refusals naming the bare pause are in `internal/shedengine/goto.go` and `internal/shedengine/run.go`.
- `orch-split`'s planned Go hub loop, when built, reads the pause record before starting ready tasks or landing.
