# Decision record — umbrella-daemon

## Goal

Every beam, the hub's prime included, gets dobby: a Go daemon with no LLM that starts the beam's ready tasks, drives their batten runs, lands them and cleans up.
It replaces the one waiting `lyx batten run` process per run.
It escalates only what needs judgement: first to the arbiter, then to the nearest live orch.
The orch stops starting, resuming or cleaning up runs, and one `lyx orch start` after a machine restart brings every run back.
In the same task, the arbiter becomes part of every shed run's stop chain, batten included.

## Scope

In:

- dobby, the beam module's daemon, and `lyx beam start`, called by `lyx orch start`, `lyx beam create`, `lyx beam promote` and `lyx adopt`.
- Batten in short passes: the Run-Shed row's InnerRun checks its child once per pass instead of waiting in one long process, and dobby schedules the passes.
- `draft` as a board task status, the default for a new task, and readiness computed by dobby from the board.
- The landing notice's `newly ready` part and the orch's start-on-notice steps go; every kind of batten notice is revised (D9).
- Automatic revive of a session stalled on an API error.
- The board-tidy Sonnet agent every N landings, spawned by dobby.
- Key-level reporting in Finalize's per-worktree config notice.
- The arbiter seam moved into shed beside the tackler, wired for every shed-driven recipe, with its keys in `shed.yaml` and per-module overrides, and a description of batten's rows (parent decision, 2026-10-25).
- The orch role stencil, the `lyx batten run` help, the module docs, `docs/overview.md` and `PATTERN.md`, with "umbrella" renamed to "beam" in every text this task touches.

Out:

- Moving batten's logic into dobby: batten keeps its rows and their code, and dobby only schedules passes.
- Changing `lyx tender`: it stays the hub-wide brake, apart from dobby (operator decision, 2026-10-25), and `lyx tender revive` stays a manual verb.
- Noticing a live strand left idle with a background fork after `lyx tender resume`: the quiet notice covers an idle run, and nothing more is built for it.
- A per-beam anchor (`umbrella-anchor`), and the darn pieces inside a beam (`loom-beam-darns`), which builds on this task's dobby.
- Note `landing-escalate-arbiter`: D8 leaves loom's landing-halt rule as it is, so that note's change stays its own.
- Starting dobby when the machine starts.
- Removing the orch's hub-wide rule section "Board tidy after a landing": that is a Bolt write through `lyx orch rules`, which the parent does once this lands; D9's dropped `landed` notice leaves the rule with no trigger, so it cannot double a tidy meanwhile.

## Decisions

### D1 — dobby is the beam module's daemon, one per beam worktree

Dobby lives in the beam module: the verbs in `internal/beamcli` and the daemon in a new told-geometry engine package (`internal/dobbyengine`), run by a hidden `lyx beam dobby` verb, the way `lyx orch watch` runs the watcher.
There is one dobby per beam worktree on its owning machine: the hub's prime, which is the root beam, on the recorded prime owner's machine, and each local pair whose origin record carries the beam mark, on its owner's machine.
It holds a lock file and a state record under its worktree's `.lyx/beam/`, both named through `internal/lyxdirs`; the state is a tolerant JSON record written through `internal/state`.
A second dobby for the same worktree exits at once on the held lock.
A dobby that reads its beam owned by another machine, at start and on each probe tick, exits.
Its creel sender is the program `DOBBY`: `ly:DOBBY` for the prime and `ly:<slug>:DOBBY` for another beam, formed through `internal/agentname`.
Its working directory is the hub root, never a worktree, and it is told absolute paths, so the teardown of a pair never runs inside it.
It registers in `internal/runproc` with a new role, `RoleDobby`, under the beam's slug, or `runproc.HubOwner` for the prime, so the pair teardown of a landed beam ends that beam's dobby.
Bound: `runproc.End` of the beam's own slug, from a cancel or owner pause of the beam's landing run, also ends its dobby; by then `BeamLandRefusal` has left the beam no members, so the dobby drives nothing, and `lyx beam start` brings it back if the beam stays.
On a changed `lyx` binary it re-execs itself between ticks, as the orch watcher does.

Rationale: the operator put beam and dobby into one module (2026-10-18), and the orch watcher is the existing shape for a detached daemon.

### D2 — `lyx beam start`, and every beam has a dobby from birth

`lyx beam start` starts, detached, the dobby of every beam worktree this machine owns whose dobby is not live, and reports for each whether it started it or found it live.
With no prime owner recorded, it records this machine first, as a batten resume does; with another machine recorded, it starts no prime dobby here and says so.
It is idempotent and runs from the prime or any beam.
`lyx orch start`, with or without `--continue`, calls it first; a failure there is a Warn on orch's envelope naming `lyx beam start`, and never stops the orch's start.
`lyx beam create` and `lyx beam promote` start the new beam's dobby as their last step, and `lyx adopt` ends with `lyx beam start`, so a dobby runs wherever the adopted runs' parent now is.

Rationale: the operator decided `lyx beam start` and that `lyx orch start` calls it (2026-10-18), and that a run's parent always has a running dobby (2026-10-25).

### D3 — which runs a dobby drives

A dobby drives every run on this machine whose parent worktree is its own.
It finds them from the local batten status files of the hub, never from the board, so a run whose task Land already pruned is still driven through Worktree-Teardown; each run's parent is the one its pair's origin record names, per `PATTERN-pair-parent`.
It watches the batten runs directory, so a run another verb starts is seen at once.
It drives only runs this machine owns, per `PATTERN-owner-machine`; the owner check inside each pass covers a run another machine takes over.
A beam's own landing run is a run whose parent is the beam's parent, so the parent's dobby drives it.

### D4 — batten runs in short passes that dobby schedules

`lyx batten run <slug>` becomes one pass: it steps the lifecycle through the Go driver as today, and the Run-Shed row's InnerRun does one check of its child per Call instead of waiting.

The yield is a new shed outcome, `Yield`: InnerRun returns it when its check ends nothing.
Shed persists nothing on a yield, so batten's status file, history and activity stay as they were, and the step-until-stop loop ends.
The Go driver reads a yield as no stop: no rule-table row, no stop report, no repair, bounce or tackler budget, and the pass exits 0.
Bound: a yield leaves the run exactly as it was, at the same row and `running`; only a producer at a wait that its own next pass re-checks may return it, and InnerRun is the only one.

A pass dobby schedules into a running batten run at Run-Shed is a continue pass (a hidden `--continue-pass` flag): it skips the resume's origin reconcile and the Open-Window re-run.
Both run on the first pass after a dobby starts and on every pass that resumes a halted or paused batten run, so ownership is reconciled after a restart, and between those InnerRun's own owner check covers it.

InnerRun's memory across checks moves to a per-run tolerant record under the row's scratch directory, so its once-per-episode, backoff, idle-bound and grace rules keep their meaning across passes.
Two rules stay per pass on purpose: the spawn confirmation, so each pass re-checks the child's strand and locks and respawns a driverless running child, and the batten-watched marker's write, which each pass makes with the pid of the dobby that schedules it, read from that dobby's state record.
The respawn of one running child is capped at three in one state episode of the child; past the cap the pass sends the driver-dead notice and respawns no more until the child's state changes.
Every other rule stated per batten process today becomes per episode, the revive of a halted child's dead driver strand included.
Bound: a file that ends a wait between passes, such as a stop report or a strand's end, is seen at the next pass, so each such wait can end up to one `notice_probe_s` later than today, and none is lost.

Dobby runs a pass of a run it drives only while batten's own state is `running`, and never while the slug's cancel marker exists.
A halted or paused batten run waits for a resume by the orch or the operator, `lyx batten run <slug>`, which is itself a pass.
It runs one:

- when the child's status file is written, seen through `internal/fswatch`;
- when batten's own status file is written while no pass of the slug is in flight, so a pass's own writes never trigger the next;
- on a probe tick, once per batten's `notice_probe_s`;
- at once when the dobby starts, and when a tender pause entry covering the run is lifted.

It never runs two passes of one slug at once, and reads a held driver lock (`shedrun.DriverLock`) as a pass in flight, so a pass left by a dead dobby is waited for and never doubled.
A pass runs in its own process group, so a dobby that dies leaves it to finish.
Bound: per run, at most one pass per `notice_probe_s` plus one per write of the child's status file, and at most one in flight.
A pass that exits non-zero is retried on the next probe tick; three failures in a row with the same error text post one notice per error text to the nearest orch, and the retries go on.

`pairteardown`'s readers of a live batten, the claim settlement and the session end before teardown, read a run as having a live lifecycle owner also when a live dobby drives it, so the gap between passes never reads as a gone batten.

`--window` and the hidden `--open-terminal` are retired, which the entry did not ask for but follows from dobby starting every run; the open_window setting alone decides the Open-Window row.
`battencli.ResumeInWindow`'s callers, `lyx adopt` and `lyx beam land`, start the run's first pass detached instead and then make sure the parent's dobby is live.
A hand-run `lyx batten run <slug>` runs one pass in the foreground and then starts the parent's dobby when it is not live.
The run removes note `batten-window-default` with `lyx board remove`, since its idea is moot.

Rationale: the operator agreed that one daemon replaces one waiting batten process per run, and that batten drives one task's lifecycle as today and knows nothing of its owner.
Short passes keep every row and notice in batten's code; persisting InnerRun's memory keeps its once-per-episode rules true when each pass is a new process.

### D5 — `draft` is a task status, and dobby starts ready tasks

`draft` is one more value of a task's `status`: not a run status, so it locks nothing.
Every board write that creates a task with no status, an upsert, a batch upsert or a merge, and `lyx board promote` of a note, give it `draft`.
`lyx batten cancel`, with or without `--remove`, leaves the task `draft` rather than clearing its status, so a cancelled task is not started again.
`lyx board set-status <slug>` with no status clears it, which is how the operator, or the orch on the operator's word, releases a task.
Batten's start guard refuses a fresh start of a draft task, naming `lyx board set-status <slug>`, so a hand-run start cannot skip the release.
The README keeps a draft task in its Ready or layer section with its After column, marked `draft` before its title.

A one-time board migration marks every open task with an empty status `draft` and records a top-level `draft_status` key in `board.json` in the same write, so tasks made by an older binary before this lands wait for a release; a board holding the key is never migrated again, and an older binary's rewrite keeps the key, per `PATTERN-tolerant-records`.

A dobby starts a task when all hold: kind task; status empty; every `depends_on` done; its parent is the dobby's worktree; its owner is this machine or none; no tender pause entry covers the hub or the slug; and fewer than `max_runs` runs it drives are in flight, halted ones included.
Starting is the run's first pass, which claims the task as batten does today.
`max_runs` is a new key of the beam module's config, `beam.yaml`, default 3, settable per worktree layer; 0 starts nothing.
A first pass that batten refuses before the claim posts one notice per refusal text to the nearest orch, and the start is retried from one probe interval, doubling up to an hour, and at once on any change of the task's board entry; the backoff is kept in the dobby's state record.
Dobby reads the local board on a write to `board.json` and on its probe tick; a release written on another machine reaches it when this machine's board is next pulled.
Bound: a task is started only from an empty status, by its parent's dobby or a hand-run `lyx batten run`, and never twice while a pass of it is in flight; across machines, only the owner machine's dobby of the parent runs.

Rationale: the operator decided readiness without batten notices (2026-10-19) and draft as a status (2026-10-25); "a new task is a draft by default" (2026-10-18) carries over to the status.

### D6 — the landing notice names no ready task

The Land row's ready snapshot, its `newly ready` routes (`LandDeps.ReadyRoutes`) and their marker lines go.
The orch role stencil drops the start steps (`lyx batten run <slug> --window`, start-on-notice and the creel-log fallback) and the window wording of its landing sections.
The orch never starts, stops or cleans up a batten run.

### D7 — stops: shed's chain for runs, the orch for dobby's own failures

Each batten pass steps through the Go driver, so a batten stop meets the rule table, the tackler and the arbiter (D8) before anything reaches the orch, and the child run's own driver does the same for its stops.
Dobby adds no rule table: the mechanical stops are settled where they are today.
Dobby's own failures, a refused start (D5) and a pass that keeps failing (D4), are not shed stops, and dobby posts them to the nearest live orch up its beam chain, the existing notice routing, one line per episode with its marker under `.lyx/beam/`.

### D8 — the arbiter for every shed run (parent decision, ly:orch, 2026-10-25)

The arbiter seam moves into shed beside the tackler: its spawn and its prompt move out of `internal/loomcli` into a shed-layer package that batten imports without linking tree-sitter, and `sheddriver`'s `Arbiter` seam is wired for every shed-driven recipe, loom, darn and batten.
Its keys, `arbiter`, `arbiter_timeout_min` and `arbiter_max_spawns`, move to `shed.yaml` with today's loom defaults.
`loom.yaml`, `darn.yaml` and `batten.yaml` each get same-named override keys in the tackler overrides' shape: an empty model-spec and a 0 timeout inherit `shed.yaml`, `arbiter_max_spawns: ~` inherits, and a set 0 turns the arbiter off for that module.
A hub layer that set a key in `loom.yaml` keeps its meaning as a loom override, an explicit 0 included.
Darn reads loom's keys today, so the config migration copies each arbiter key a hub layer sets in `loom.yaml` into `darn.yaml` where that layer's `darn.yaml` sets none, keeping darn's behaviour.
The arbiter prompt names the recipe's rows through a per-recipe description, as the tackler's prompt does, and batten's description names its rows and what each does.

`internal/arbiterstop`'s eligibility gains a rule per recipe: for batten, a stop the tackler escalated is eligible unless it is a hand-back (a landing halt, a spent budget or an unmet precondition), which goes to the run's parent as today.
Loom's and darn's rules are unchanged.

Batten's Land and Worktree-Teardown failure notices now wait for the stop chain: the row notifier posts once the batten run's Go driver hands the stop back, with its stop report, instead of on the row's failure, so the orch is told only what the tackler and the arbiter did not settle.

Rationale: every shed run must be able to spawn the arbiter, to take load off the orch (operator).

### D9 — batten's notices, revised now that dobby runs

- `landed`: dropped; a landing that succeeds needs nothing from the orch, and the board tidy (D10) takes over what the orch did after a landing.
- A child that halts and whose driver handed the stop back, an awaiting child with a parent notice, the busy-pane draft stall, the owner-elsewhere notice, and Land and Worktree-Teardown failures (held per D8): kept, since each needs judgement.
- driver-dead: sent when the respawn cap (D4) is spent, or a revive of a halted child's strand failed.
- api-error: the pass first sends tender's revive line to the stalled session through the conditioned send, once per episode; the notice goes when the same episode still stands at the next probe check, or when the third revived episode in a row comes with no write of the child's status file between them.
- quiet: kept, informational.

Every kept notice keeps its text, routing and once-per-episode marker.

### D10 — the board tidy every N landings

Dobby counts a landing at the end of a pass whose batten history holds Land Done, keyed by the run in its state record, so a restart neither misses nor double counts one.
At every `board_tidy_every` landings (a new `beam.yaml` key, default 5, 0 off) it spawns the board-tidy agent with the landed slugs since the last tidy: a fresh Sonnet session at medium effort, an interactive shuttle run in a window of the beam's reed session, briefed from a stencil read at call time through `internal/stencilstore`.
The brief is today's orch rule: read each landed commit and its run's decision record against the board's notes, through `lyx board` only, make each obvious amendment, and report each note a landing superseded or changed in scope or design.
Its parent directive names the nearest live orch, so a question goes to an LLM that can answer it.
When its report exists, dobby posts its path to that orch, which puts each removal or scope change to the operator.
The tidy in flight is recorded in the state record, so a restart or re-exec spawns no second one.
Bound: one tidy agent at a time per dobby, ended through the shuttle stop verb after its report or after `board_tidy_timeout_min` (a new `beam.yaml` key, default 30).

### D11 — key-level config notice at Finalize

Finalize's per-worktree config notice compares keys, through `internal/configengine`, at the fork base, the task tip and the parent tip.
It is silent when the tip changes no key, as for an empty file or comments only, and when the parent already holds the tip's value.
Otherwise it names each changed key with its base, tip and parent values, an absent key or file reading `absent`.
The operator called this a job for the Go run driver; it stays in Finalize, the mechanical Go step that holds the three commits under the parent lock, rather than in dobby.

### D12 — tender stays the brake

Dobby starts no task while a pause entry covers the hub or the slug.
Inside a pass, batten's existing pause handling holds every spawn, resume and revive, and InnerRun yields rather than returning Done for a done child while a pause covers the slug, so no run enters Land under a pause.
When an entry is lifted, dobby runs a pass of every running run it covered at once (D4), whose respawn, revive and driver-dead checks show whether the run came back.

### D13 — dobby's wakeups

A dobby wakes on an `internal/fswatch` event and on its probe timer, and coalesces events to at most one tick a second.
With no run in flight, its timer backs off by doubling up to sixty seconds, and any event resets it.
Where fswatch fails to open, as on WSL or through a junction, the timer alone drives it at the probe cadence.

### Added after Discussion (parent, 2026-10-25): The prime is the root beam, on the same code path as every other beam

Decision: The prime worktree is a beam like every other beam: dobby and the beam code treat it as the root beam, with no special path for it. The only differences: it has no parent, so `lyx beam land` never lands it; it needs no project entry or beam mark, being a beam by being the top worktree; and the hub-wide duties stay with it (the board on Bolt, hub config, the landing deploy and the hub's prime owner).

Rationale: Operator, 2026-10-25: the top worktree is equivalent to every other beam except that it has no parent.

## Constraints

- `PATTERN-shed-driver`: the step loop stays in `internal/sheddriver`, and dobby calls no step body; it starts `lyx batten run` processes, registered in `internal/runproc` and logged through `internal/logger`, a yield at Debug.
- `PATTERN-durable-vs-ephemeral-state`: dobby's lock, state and markers sit under `.lyx/beam/`, named only in `internal/lyxdirs`.
- `PATTERN-agent-name`: `DOBBY` is the beam module's constant.
- `PATTERN-cwd-resolution`: `internal/dobbyengine` is told every path and never imports `internal/lyxcwd`.
- `PATTERN-config-layers`: `beam.yaml` and the arbiter keys are read only through `configengine`, and D8's copy into `darn.yaml` runs as a config migration through `hubreconcile`.
- Running task `discussion-scope-in-lenses` also edits the orch role stencil and renames `Merge-In` to `Parent-Align` in the landing texts `arbiterstop` and `sheddriver` hold; whichever lands second takes the other's wording in those files.
- `lyx batten run` already resumes from disk with every row idempotent, and its notice markers already survive a restart (`internal/battenshed/doc.go`), which is what a pass relies on.
- `boardengine`'s store accepts one on-disk version, so D5's migration records a key rather than raising the version, which an older binary on another machine would refuse to load.

## Auto-mode assumptions

- No operator answered: every choice in the question ledger was picked by the writer.
- "batten and dobby included" in the parent's arbiter decision is read as: the arbiter reaches every stop of a run dobby drives, since dobby drives batten through the Go driver; dobby's own failures are not shed stops and go to the orch (D7).

## Open risks

- The InnerRun memory record must cover every field that today's in-process wait carries across checks; a field left out turns a once-per-episode rule into once per pass.
- A pass per probe tick per run spawns many short `lyx` processes; D4's bound caps them, at the cost of process starts (see note `spawn-audit-log`).

## Acceptance criteria

- `lyx beam start` starts a dobby for the prime and each owned beam and reports live ones as found; a second start changes nothing; it starts no prime dobby where another machine is the prime owner; `lyx orch start` runs it first and still starts when it fails.
- A second dobby for one worktree exits on the held lock, and a dobby whose beam another machine owns exits.
- A beam made by `lyx beam create` or `promote`, and the parent of a run `lyx adopt` took, has a live dobby when the verb returns.
- A task with an empty status, all dependencies done and the dobby's worktree as parent is started; a draft task, one with an open dependency, one under a covering pause, one owned elsewhere and one past `max_runs` are not.
- A new task from any creating write and a promoted note get `draft`; a cancelled task is left `draft`; a hand-run start of a draft is refused naming `lyx board set-status`; the migration marks statusless open tasks once.
- A refused start is retried on its backoff and at once on a change of the entry, with one notice per refusal text.
- A batten pass yields at Run-Shed with the status file untouched and no cap or budget spent; the next pass resumes it; its own writes trigger no pass.
- A continue pass runs no origin reconcile and no Open-Window re-run; the first pass after a dobby start runs both.
- A halted or paused batten run gets no pass from dobby, nor does a slug with a cancel marker.
- A notice sent once per episode by today's long wait is sent once across many passes; a driverless running child is respawned by a pass, at most three times per child state episode, then noticed.
- `pairteardown`'s claim settlement and session end read a run between passes as owned while its dobby lives.
- A pass is never run twice at once for one slug, a held driver lock left by a dead dobby is waited for, and a restarted dobby runs a pass of every run in flight, a run Land already pruned included.
- Under a covering pause no run enters Land; after the lift, every covered running run gets a pass at once.
- The landing notice names no ready task, and no `landed` notice is sent on success.
- An api-error stall gets one revive line per episode, and the notice when the episode stands at the next probe or on the third revived episode in a row.
- A batten stop the tackler escalates reaches the arbiter, and a batten landing halt does not; a Land failure is noticed only after the driver hands it back; loom's and darn's eligibility is unchanged.
- The arbiter keys resolve from `shed.yaml` with module overrides; an existing `loom.yaml` value keeps its meaning for loom and, through the copy, for darn.
- Every `board_tidy_every` landings dobby spawns one tidy agent, counted once per landing across a restart, and posts its report path to the nearest orch; 0 spawns none.
- Finalize's config notice is silent for a comments-only change and for a key the parent already holds, and names base, tip and parent values otherwise, `absent` included.
- A pair teardown of a landed beam ends its dobby.
- Dobby wakes at most once a second and backs off while idle, and falls back to its timer when fswatch fails to open.

## PATTERN changes

- `PATTERN-dobby-starts-runs` — adds: Starting or driving a board task's run: its parent's dobby, from an empty status, in batten passes; a hand `lyx batten run` is one pass.
- `PATTERN-shed-stop-chain` — adds: A shed run's stop: rule table, tackler, arbiter, then parent, for every recipe; arbiter keys in shed.yaml with module overrides.
