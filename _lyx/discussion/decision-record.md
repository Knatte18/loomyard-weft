# Decision record — twb-d0-ha: hibernate the machine on a trigger

## Goal

The operator, from the phone through the orch or from a shell, queues a hibernate of the machine that fires at once, when one named task lands, or once no batten run in the hub is stepping.
No LLM session runs the hibernate: the session's part ends at writing the request, and lyx's Go code pauses the hub, confirms, hibernates and resumes.
The board entry's pause and resume shortcuts land in the same task.

## Scope

In:

- A new module `lyx hibernate` with the verbs `request`, `cancel` and `status`, plus a hidden daemon verb `watch` that `request` spawns detached.
  `request` with no trigger flag is the board's plain `lyx hibernate` verb.
- A hibernate request record: one JSON file under the board-level `.lyx`, beside the pause record.
- The trigger forms: now (the default), `--after <slug>` and `--when-idle`.
- A seam in batten's Land row that marks a matching `--after` request as triggered.
- A scope-exact resume in `tenderengine`, which lifts the hub entry alone (D8).
- The waiter's sequence: wait for the trigger, take a hub-wide pause, confirm, hibernate, then resume after the wake.
- Orch notices, through `orchengine`'s notice queue, when a request fails and after a wake.
- The shortcuts `p: [tender, pause]` and `r: [tender, resume]` in `internal/shortcutcli/shortcuts.yaml`.
- Module docs: each new package's `doc.go`, `tenderengine`'s doc for the new resume, the module table in `docs/overview.md`, and a sandbox-suite entry.

Out:

- Stopping or holding background work (forks, background shells, Monitors) under a pause, and any change to what a pause counts as busy: `pause-background-work` owns these.
- Coordinating hibernate across several hubs on one machine; a request covers its own hub.
- Any platform but Linux.

## Decisions

### D1 — Two new packages, `internal/hibernateengine` and `internal/hibernatecli`

The engine holds the request record's types, the trigger checks, the waiter's sequence and the confirmation predicate, over a `Deps` of seams: pause, scope-exact resume, session and network readings, the land lock probe, batten run readings, the hibernate command, the orch notice, the clock and the sleep watch.
It imports neither `tenderengine` nor `lyxcwd`, and is told every path (`PATTERN-told-geometry`).
The CLI is a cobra subtree under the root in `cmd/lyx/main.go` (`PATTERN-cli-cobra`) that resolves the hub with `lyxcwd.Resolve` and fills `Deps` over `tenderengine`, shuttle, reed, batten's status files and `orchengine`.
It runs from any worktree of the hub, like `lyx tender`, so cliwire does not serve it, for the reason `tendercli`'s doc gives.

Rationale: the same engine and CLI split as `tenderengine`/`tendercli`; seams keep every test off the real machine.

### D2 — The request record

One file, `hibernate.json`, under the board-level `.lyx`, its path declared through `internal/lyxdirs` and `internal/hubgeom` like the pause record (`PATTERN-lyxdirs-single-declarer`, `PATTERN-durable-vs-ephemeral-state`).
It holds at most one request: its trigger (`now`, `after` with a slug, or `when-idle`), the caller, the creation time, the waiter's pid and start time, a state, the hub entry the waiter took (its creation time, empty when the waiter used another caller's entry), and the last failure reason.
The states are `waiting`, `triggered`, `pausing`, `confirming`, `hibernating`, `resuming`, and the terminal `done`, `failed` and `cancelled`; the first six are live.
A terminal request stays until the next `request` replaces it, so `status` reports how the last one ended.
Every read-check-write runs under one lock beside the file, through `internal/state`, so an unknown key survives a rewrite (`PATTERN-tolerant-records`).

Rationale: the hub-level ephemeral directory is where the pause record lives, and every reader of the request is hub-level.

### D3 — Verbs and callers

- `lyx hibernate request [--after <slug> | --when-idle]` writes a `waiting` request, or a `triggered` one for the default trigger, under the record lock, spawns the waiter, and reports the request and the waiter's pid.
- `lyx hibernate cancel` moves a request in `waiting`, `triggered`, `pausing` or `confirming` to `cancelled`, and lifts the waiter's recorded hub entry through the scope-exact resume.
  A request in `hibernating` or `resuming` with a live waiter is refused, saying the hibernate is under way and naming `lyx hibernate status`.
  A request in either state whose waiter is dead is cancelled the same way, which is the recovery for a waiter killed mid-sequence.
  With no live request it succeeds with nothing cancelled.
- `lyx hibernate status` reports the request record, whether its waiter is alive, and, while `confirming`, what still withholds confirmation.
  It writes nothing.
- `lyx hibernate watch` is the hidden daemon verb.
  It acts only on the live request in the record and exits when there is none or another recorded waiter is alive.

Callers are read as `tendercli` reads them, from the strand-name variable reed injects.
The orch and an external caller may run every verb.
An agent's `request`, `cancel` and `watch` are refused with the way forward of asking its parent session (`PATTERN-refusal-way-forward`, `PATTERN-no-denied-recovery`).
`status` takes no caller check.

Bound: `request` can only queue a hibernate of the machine that runs the hub, and never pauses or hibernates in its own process; `watch` adds nothing a live request did not already ask for.

Rationale: a session that runs the hibernate is left hanging mid-call when the machine goes down (board, 2026-10-14).

### D4 — Request refusals

`request` refuses, each naming the way forward and landing its spec row and test (`PATTERN-refusal-way-forward`):

- a platform other than Linux, or a machine with no `systemctl` on `PATH`, saying hibernate is unsupported here;
- a live request, naming `lyx hibernate status` and `lyx hibernate cancel`;
- `--after` and `--when-idle` together;
- `--after` naming a slug with no pair or no batten run, listing the slugs that have one, as `lyx tender pause` does.

A run that is paused, halted or waiting on the orch, such as at PR-Gate, is accepted, since it can still land.

### D5 — The waiter

`request` spawns its own executable as `lyx hibernate watch` through `proc.Detach`, the way `orchcli` spawns the orch watcher, and logs the spawn (`PATTERN-spawn-observability`).
The spawn drops the strand-name variable from the waiter's environment, so tender never reads the waiter as the orch.
The waiter records its pid and start time in the request under the record lock.
While `waiting` it polls each second and backs off to five seconds while nothing changes (`PATTERN-daemon-wakeups`):

- for `--when-idle`, the trigger is met when no batten run in the hub reads running; with none running at the request, that is at once;
- for `--after`, the trigger is met when the Land seam (D6) has marked the request `triggered`;
  the slug's pair gone with the request still `waiting` fails the request, naming the slug.

The waiter re-reads the request under the record lock before each state move, and a request no longer in the state it left ends the waiter with no further step, so a cancel at any live state up to `confirming` stops it before the pause or before the hibernate command.
A waiter that finds the request cancelled once its pause returns lifts the entry it took, since `cancel` may have run before the entry was recorded.
A waiter that died leaves the request live, `status` reports it dead, and the way forward is `cancel` then `request`.

### D6 — Batten's Land row marks an `--after` request

After the Land row's deploy and board prune are done, a seam reads the request and, when it is `waiting` and `after` this row's slug, moves it to `triggered` under the record lock.
The request path is told to the row through `LandDeps` (`PATTERN-told-geometry`).
The seam runs on a successful Land only; a Land that halts leaves the request `waiting`.
A failure to read or write the request is logged and never fails the Land.

Bound: the seam changes one field of the request record and nothing else; it never pauses, interrupts or hibernates, so a request never holds a Land.

Rationale: the board names batten after Land, deploy included, as the firing point, so nothing is mid-write.

### D7 — Pause and confirmation

On `triggered` the waiter first waits for the hub's land lock to be free, then moves to `pausing` and takes a hub-wide pause through the pause seam (`tenderengine.Pause` with an empty slug and the default grace), with the caller `hibernate`, which the entry records.
When a live hub entry another caller took already stands, the waiter uses it, records no entry of its own, and lifts none later.
The keys any pause sends are the pause's own, through shuttle's guarded interrupt; the waiter types no text and sends no key outside that seam.

It then moves to `confirming` and polls each second for up to ten minutes of wall time, until all of these hold at one reading:

- a hub entry is live;
- no session of any task pair or of the prime, the orch included, reads busy with a running turn or with background work, by tender status's readings;
- no network operation is in flight (`pauserecord.ListNetOps`);
- the land lock is free.

The land lock is probed by a try-acquire released at once.
A hub entry gone at a reading, because the operator ran `lyx resume`, fails the request, since the resume tells the waiter the operator wants the runs going.
The orch has no exemption: the hub entry has the orch watcher interrupt a busy orch at the grace end, and an operator chat during confirmation withholds it until that turn ends, since a hibernate mid-turn leaves the orch's call hanging.
This settles `pause-background-work`'s orch point for this module; that entry still owns background work, which withholds confirmation until it lands.

A timeout fails the request with the sessions and operations that withheld it, lifts the waiter's recorded entry, and queues one orch notice naming them.

### D8 — Scope-exact resume, hibernate and wake

`tenderengine` gains a resume of the hub entry alone: it lifts the live hub entry when its creation time matches the one told, leaves every slug entry live, and gives each session the hub entry owned the same handling `Resume` gives it today, including a move to a slug entry that still covers the session.
Every lift in this module goes through it.

On confirmation the waiter moves to `hibernating`, logs the spawn and runs `systemctl hibernate`.
A non-zero exit fails the request with the command's output, lifts the waiter's entry and queues an orch notice.
On a zero exit the waiter polls `pauserecord.SleepWatch` each second and moves to `resuming` at the first reading that sees a sleep end.
It then lifts its entry, moves the request to `done`, and queues an orch notice naming the hibernate and wake times.
No sleep seen within ten minutes of awake time after the command's exit fails the request the same way.

Rationale: the pause was the waiter's means to a safe hibernate, not the operator's; leaving it live would hold every run until someone typed `lyx resume`.

### D9 — Shortcuts `p` and `r`

`internal/shortcutcli/shortcuts.yaml` gains `p: [tender, pause]` and `r: [tender, resume]`, beside `a`.

Rationale: the board proposed these names, they shadow no top-level module, and a shortcut lands by a commit to that file.

## Constraints

- `PATTERN-orch-pane-single-writer`: the waiter reaches the orch only by queuing an `orchengine` notice.
- `PATTERN-daemon-wakeups`: the waiter's files join the declared list in `cmd/lyx/daemonwakeups_test.go`, per the entry's background file.
- `PATTERN-log-levels`: the waiter's polls log by the logger's loop policy.
- `PATTERN-sandbox-coverage`: the sandbox suite exercises the module with the hibernate seam faked, or excludes it with the reason that it hibernates the machine.
- `PATTERN-test-speed`, `PATTERN-test-isolation`: tests drive the hibernate command, clock and sleep watch through seams and never run `systemctl`.
- `PATTERN-documentation-lifecycle` and the repo's docs rule: the docs in Scope land in the same commit as the code.
- `cmd/lyx/testdata/test-budget.yaml` goes under a card's `Edit:` if its new tests push a package past its row.

## Auto-mode assumptions

- "Every instance" in the board's direction reads as every session in the hub (Scope).
- The waiter resumes its own pause after the wake (D8); the board leaves resume unsaid.
- `--when-idle` counts a run halted or blocked waiting on the orch as done (D5).
- The ten-minute confirmation bound (D7) matches the bound `pause-background-work` records.
- The shortcuts are added (D9); the board left the names to the operator.

## Open risks

- Until `pause-background-work` lands, a session with live background work makes confirmation time out (D7), so a request taken while Webster forks run fails rather than hibernates.
- The waiter's pause can catch another run in Land's waits and halt it blocked at Land, which then needs the orch after the wake.
- A deploy in the triggering Land replaces the `lyx` binary under a running waiter; on Linux the waiter keeps running its deleted file, and tender status lists it under the replaced-binary processes.
- After a wake the network may not be up when the pause lifts; held calls then fail as any network failure does, and `lyx tender revive` covers a session stalled on an API error.

## Acceptance criteria

- `request` with no trigger pauses the hub, confirms, runs the hibernate seam once, and after a sleep reading lifts its entry, ends `done` and queues the wake notice.
- `--after <slug>` stays `waiting` until that slug's Land row succeeds, then proceeds; a Land that halts leaves it `waiting`; the pair removed with the request `waiting` fails it.
- `--after` is accepted for a run that is paused, halted or waiting on the orch.
- The Land seam marks only a `waiting` request for its own slug, and a request read or write failure never fails the Land.
- `--when-idle` triggers once no batten run reads running, at once when none does, and not while one does.
- Each refusal in D4 fires with its way forward; an agent caller is refused `request`, `cancel` and `watch` but may run `status`.
- Two concurrent `request` calls leave one live request and one live waiter.
- Confirmation withholds while a session, the orch included, reads busy with a turn or with background work, while a network operation is in flight, or while the land lock is held, and passes once none holds.
- A hub entry lifted during confirmation fails the request with no hibernate.
- A confirmation timeout fails the request with what withheld it, lifts only the waiter's hub entry with every slug entry left live, and queues one orch notice.
- A hub entry another caller took is neither replaced nor lifted by the waiter.
- A failing hibernate command fails the request, lifts the entry and queues a notice; no sleep seen within the bound does the same.
- `cancel` during `waiting`, `triggered` or `pausing` ends the waiter with no hibernate and no entry left live from it; during `confirming` it lifts the waiter's entry and no hibernate runs; during `hibernating` with a live waiter it is refused.
- A waiter dead mid-sequence reads dead in `status`, and `cancel` then `request` recovers.
- A second waiter for a request whose recorded waiter is alive exits.
- The scope-exact resume lifts the hub entry alone and leaves slug entries and their sessions untouched.
- A request record with an unknown key keeps it across a rewrite.
- The waiter's polls back off while nothing changes and never wake more than once a second.
- `lyx p` runs `lyx tender pause` and `lyx r` runs `lyx tender resume`.

## Notes for the plan writer

- `internal/tenderengine` and `internal/tendercli` hold the pause, resume and status sequences and the caller check this module reuses.
- `internal/orchcli`'s spawn of the orch watcher is the nearest precedent for a detached daemon verb.
- `internal/battenshed` and `internal/battencli` hold the Land row, its seams and the land lock.
- `internal/pauserecord` holds the sleep watch, the network operation listing and the record conventions the request file follows; `internal/hubgeom` shows the hub-level path.
