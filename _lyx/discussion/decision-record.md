# Decision record — twb-d0-hf: hibernate the machine on a trigger

## Goal

Let the operator, usually from the phone through the orch, queue a machine hibernate in lyx: immediately, once a named task lands, or once every batten run in flight has landed.
No LLM session runs the hibernate.
A detached Go waiter pauses every lyx hub on the machine, confirms the pause, runs `systemctl hibernate`, and resumes its own pauses after wake.
Add the `p` and `r` shortcuts for `lyx pause` and `lyx resume`.

## Scope

In:

- A new `lyx hibernate` cobra group with `request`, `cancel` and `status`, plus one hidden verb the waiter runs as.
- A new engine package behind it, over seams, and a CLI package that wires it.
- A machine-level hibernate request record under the user state directory.
- Hub discovery from the machine's lyx processes.
- The pause, confirm, hibernate, wake and resume sequence, including its failure paths and an orch notice on failure.
- Exporting `internal/termwindow`'s WSL test and `internal/standalonestate`'s state-home root, each as a function its own package owns.
- A `## hibernate` section in `contracts/specs/refusal-spec.md` with a row per new refusal, as `## pause` has for tender.
- `p: [tender, pause]` and `r: [tender, resume]` in `internal/shortcutcli/shortcuts.yaml`.
- One new PATTERN entry, the package `doc.go` files, the `docs/overview.md` module row and shortcuts line, and the sandbox exclusion.

Out:

- Making background forks, shells or Monitors stop under a pause, or counting a session busy only with background work as idle; that is `pause-background-work`'s scope.
- Any change to batten's recipe or its Land row.
- Any change to tender's pause, resume or revive semantics.
- Platforms other than native Linux.
- Suspend-to-RAM, shutdown, or a scheduled time trigger.

## Decisions

### D1. Verb surface

`lyx hibernate` is a new top-level cobra group (PATTERN-cli-cobra) with three operator verbs:

- `request [--after <slug> | --all]`: with no flag, the trigger is met at once; `--after <slug>` waits for that slug's batten run in the requesting hub to land; `--all` waits for every batten run in flight in the requesting hub at request time to land.
  `--after` and `--all` together are refused.
- `cancel`: withdraws the request.
- `status`: reports the request, its trigger, its phase and reason, the hubs it paused, any run its trigger waits on that is halted or held, and whether its waiter process is alive; it writes nothing.

A hidden verb is the waiter's own entry point, never named in help or to agents.
`request` writes a fresh random token into the request record and passes it to the waiter it spawns.
Bound: the hidden verb applies D2's caller rule, and acts only on a request record that is non-terminal and carries the token it was given; on any other record it exits at once and changes nothing.
So it can only run the sequence of an existing request, D5 to D9, and never start a hibernate of its own.

Every verb resolves the hub with `lyxcwd.Resolve`, as `internal/tendercli` does, and not through `cliwire`, for the same dependency reason the tendercli doc gives.

### D2. Callers

The orch and an external caller may run `request` and `cancel`.
An agent, read from the strand-name variable as tendercli's `callerOf` reads it, is refused with the way forward of asking its parent session.
`status` takes no caller check.

Rationale: an agent hibernating the machine would leave every other run stopped without its parent knowing, and tender's caller rule sets the precedent.
Bound: like tender's check, this guards against a mistaken call; the agents' settings already deny the machine sleep commands and any command naming the strand variable.

### D3. The request record is machine-level, one per machine

The request lives in one JSON record under the user state directory, `$XDG_STATE_HOME/lyx/` falling back to `~/.local/state/lyx/`, taken from a new exported state-home function in `internal/standalonestate`.
`standalonestate.Derive` stays `cliwire`'s alone (PATTERN-cliwire-sole-wiring); only the root it builds on is exported.
The record holds:

- the requesting hub;
- the trigger;
- the phase: `waiting`, `pausing`, `confirming`, `hibernating`, `done`, `failed` or `cancelled`;
- the reason for a terminal phase;
- each hub the waiter is about to pause, marked `created` or `already paused` once Pause returns;
- the waiter's token, pid and start time.

Writes go through `internal/state` under a lock beside the file, and unknown keys survive (PATTERN-tolerant-records).
Every check a verb makes against the record, and the write that follows it, run under that one lock, so two concurrent `request` calls cannot both pass.

A `request` while a request is non-terminal is refused, naming `lyx hibernate cancel`, with one exception: a `waiting` request whose waiter is not alive gets a new waiter, under a new token, and `request` reports that it did.
A terminal record is replaced by the next `request`.

Rationale: the request spans hubs, so no hub's `_lyx` or `.lyx` can own it.
PATTERN-durable-vs-ephemeral-state governs files in a repository, and this file lives in none.

### D4. Hub discovery from live lyx processes

The hubs to pause are:

- the requesting hub;
- every hub that is the resolved hub root of the cwd of a running process whose executable is a lyx binary, live or replaced.

The CLI gives the engine discovery as one seam: `internal/proc`'s process listing, the hub resolution through `lyxcwd.Resolve`, and each hub's geometry, so the engine resolves nothing itself (PATTERN-cwd-resolution, PATTERN-told-geometry).
A cwd that resolves to no hub is skipped, and so is a hub under `internal/lyxtemp`'s run root, which is a test fixture.
Discovery runs when the trigger is met, and again at each confirmation poll; a hub found later is paused and confirmed like the first ones.

Rationale: a hub with no lyx process running has no session to pause, and a scan has no registry to keep in step or to go stale.
Hibernate is Linux only (D10), where the listing is supported.

### D5. Trigger

- No flag: met at once.
- `--after <slug>`: the slug must have a batten run in the requesting hub that is still stepping, or `request` is refused, listing the slugs that have one.
  The trigger is met when that run has landed: it reached its terminal, past Land, its deploy and Worktree-Teardown.
- `--all`: the set is every batten run in the requesting hub still stepping at request time; with an empty set, `request` is refused, naming the no-flag form.
  The trigger is met when each has landed.
- `--after` and `--all` are refused while a live hub pause entry covers the requesting hub, naming `lyx resume` or the no-flag form, since no run moves under it.

The waiter reads the batten runs' status through the owner of their paths and records (PATTERN-shed-run-directory), told to the engine as a seam, read-only.
A batten status file that is gone reads as landed: Worktree-Teardown removes the pair after Land, and a pair removed by hand also ends the run.

A run that halts or fails does not meet the trigger: the request stays `waiting`, and the waiter queues one orch notice for the requesting hub naming the run, its stop, and the ways forward `lyx hibernate cancel` and `lyx hibernate request` with no flag.
A run held by `lyx batten pause` or a slug pause keeps the request waiting; `status` names it as held.
A run started after the request is not waited for; the pause in D6 holds it.

Rationale: the board's trigger is a landing, and past the terminal nothing in the task's run is mid-write.
A halted run may still move: the orch, the operator, or the tackler `shed-tackler` plans for batten can resume it, so hibernating on a halt would put the machine down under a run that was about to continue.

### D6. Land lock, then pause, on every discovered hub

When the trigger is met, the waiter first takes each discovered hub's land lock, the one batten's Land row holds for its whole landing, through that lock's owner, told as a seam.
A contended lock is retried until it is free, so a Land in flight finishes before any pause goes in; a hub pause during Land's waits would halt the landing at Land instead.
The waiter holds every land lock it took until it ends, on every exit path.

It then adds a hub-wide pause entry on each hub through `tenderengine.Pause` in-process, with that hub's geometry and the default grace.
Its caller is `hibernate`, a record label only, which is never a valid strand name, so Pause skips no session for it.
Before each call, the waiter records the hub in the request; after it, the hub is marked `already paused` when Pause reports it so, and `created` otherwise.
A hub that already holds a live hub entry gets no new entry and is never lifted by the waiter.
The orch of each hub, the requesting hub's included, is interrupted by its own watcher at the grace end, as for any hub pause.

Rationale: the operator's direction asks for a pause like `lyx pause` on every instance; reusing tender keeps one pause sequence and its Esc rules.
Recording before the call means a waiter that dies mid-call leaves no entry the record does not name.

### D7. Confirm, then hibernate

After the pauses, the waiter polls each hub's tender status until, at the same poll, on every hub, every session reads not busy and no network operation is in flight.
The orch counts like any other session; its watcher's grace-end interrupt is what ends its turn.
When the condition holds, the waiter runs `systemctl hibernate`.
The confirmation is bounded at ten minutes; past the bound the request fails (D8).

A session that is busy only because of background work counts as busy.
`pause-background-work` owns that rule, and any change to how the orch's own turn counts.

Bound: the waiter never hibernates on a confirmation that did not pass, and never waits past the ten-minute bound.

### D8. Failure, cancel and operator resume

Each of these ends the request, and in every case the hubs marked created (D6) are resumed through `tenderengine.Resume` and the land locks are released:

- `tenderengine.Pause` fails on any hub: phase `failed`.
- The confirmation bound passes: phase `failed`, naming the sessions that stayed busy.
- `systemctl hibernate` exits non-zero: phase `failed`, with its output in the reason.
- No sleep of a minute or more is seen within two minutes of a zero exit: phase `failed`.
- `lyx hibernate cancel` in `waiting`, `pausing` or `confirming`: phase `cancelled`.
  Cancel writes the phase under the record lock, and the waiter sees it at its next wake and stops.
- A pause entry the waiter created is no longer live during `pausing` or `confirming`, because the operator ran `lyx resume`: phase `cancelled`, reason "resumed by the operator".

`cancel` in `hibernating` is refused as too late, since `systemctl` has run.

On `failed`, the waiter queues one notice for the requesting hub's orch through `orchengine`'s notice queue, naming the reason (PATTERN-orch-pane-single-writer).
`done` and `cancelled` queue no notice; `status` shows them.

A waiter that dies leaves its record as it was, and nothing notifies anyone, since the waiter is the only notifier.
`status` then shows the waiter as not alive and names the entries the record holds; `lyx tender status` shows each as a live hub entry with caller `hibernate`.
In any non-terminal phase with a dead waiter, `hibernating` included, `cancel` resumes every hub the record names, except one marked already paused, and records `cancelled`; the dead waiter's land locks were freed with its process.

Rationale: a hibernate that does not happen should leave the hubs as they were, and the orch should learn it, since the operator is likely away.

### D9. Wake and resume

The waiter takes `pauserecord.SleepWatch` readings with `pauserecord.MonotonicAwake` from just before `systemctl hibernate` runs.
The first reading that shows a sleep ends the hibernate: the waiter resumes the hubs marked created, releases the land locks, marks the request `done`, and exits.
A session stalled on an API error after the wake is left to `lyx tender revive`.

Rationale: the operator already hibernates with runs live and expects them to carry on after wake, and `MonotonicAwake` stops during suspend on Linux.

### D10. Platform

`request` refuses up front on any GOOS but Linux, and on WSL, through the WSL test exported from `internal/termwindow`, the one place that test lives.
Each refusal says hibernate is supported only on native Linux, and that the way forward is `lyx pause` and then hibernating the machine, or the Windows host, by hand.
The hibernate command is `systemctl hibernate`, run through one seam the engine is told.

Rationale: the operator runs lyx only on Linux, and `systemctl hibernate` inside WSL cannot hibernate the Windows host.

### D11. Shortcuts `p` and `r`

Add `p: [tender, pause]` and `r: [tender, resume]` to `internal/shortcutcli/shortcuts.yaml`, beside `a`.

Rationale: the entry proposes these names; nothing else claims them, and they follow `a` beside `attach`.

### D12. New PATTERN entry

Add `PATTERN-hibernate-single-site`, marked `(test)`: putting the machine to sleep happens only in the hibernate waiter, after a confirmed pause; no other production code and no stencil runs a machine sleep command.
A test scans production Go files and stencils for a `systemctl` sleep verb outside the hibernate engine and fails on one.

Rationale: the operator's direction that no LLM session hibernates the machine is a cross-cutting invariant.

## Constraints

- PATTERN-told-geometry: the engine is told every path and seam: hub discovery and geometries, the request record path, batten status reads, land locks, the status reader, pauser and resumer, the requesting hub's orch notice paths, the hibernate command, the waiter spawn, the clock and the awake clock; it imports no `lyxcwd`.
- PATTERN-daemon-wakeups: the trigger wait and the land-lock wait wake at most once a second and back off while nothing changes; the confirmation poll reads state records and no pane, so it wakes once a second; the waiter's file joins the declared list in `cmd/lyx/daemonwakeups_test.go`.
- PATTERN-orch-pane-single-writer: the waiter never types into an orch pane; it queues `orchengine` notices.
- PATTERN-guarded-interrupt and PATTERN-pane-text-injection: the waiter types nothing and sends no key itself; tender's pause and resume do.
- PATTERN-spawn-observability: the waiter spawn, the `systemctl` spawn and the wait on it are logged through `internal/logger`; the waiter spawn is a seam, so no test re-execs the binary.
- PATTERN-refusal-way-forward and PATTERN-no-denied-recovery: every refusal names a way forward the caller is allowed to run, with its row in `refusal-spec.md`.
- PATTERN-tolerant-records: the request record is rewritten via `internal/state` and never refuses an unknown key.
- PATTERN-lyx-temp-root and PATTERN-test-isolation: tests put the request record under the test's temp root, never the real user state directory.
- PATTERN-test-speed: inject clocks; no test runs `systemctl`, sleeps the machine, or reaches an LLM.
- PATTERN-sandbox-coverage: the sandbox suite excludes hibernate, with the reason that it puts the machine to sleep.
- PATTERN-documentation-lifecycle and CLAUDE.md "Docs land in the same commit": new `doc.go` files, the `docs/overview.md` row and shortcuts line, and the PATTERN entry land with the code.
- Dependency: until `pause-background-work` lands, any live Webster run or driver wait loop on the machine keeps confirmation from passing, so the verb meets its goal only with that entry landed; build it first, or ship knowing requests fail at the ten-minute bound while such runs are live.
- Test budget: a package past its row in `cmd/lyx/testdata/test-budget.yaml` lists that file under `Edit:`.

## Auto-mode assumptions

- "Every instance" means every lyx hub on the machine (D4).
- The board's "batten after Land" is met by the landing trigger (D5), not by batten code.
- A halted or failed run does not meet the trigger; it gets an orch notice (D5).
- The waiter resumes its own pauses after wake (D9).
- The confirmation and no-sleep bounds (D7, D8).
- The shortcut names `p` and `r` (D11).
- One request per machine (D3).

## Open risks

- `systemctl hibernate` may need privileges or swap configuration the machine lacks; the request then fails with `systemctl`'s output, and the hubs are resumed.
- A hub running a different lyx build than the waiter's is paused through the waiter's in-process tenderengine; a record format change between builds is covered only by PATTERN-tolerant-records.
- A hibernate whose power-off lasts under a minute reads as no sleep, past `SleepTolerance`, so the request records `failed` and sends a notice though the machine did sleep; the hubs are resumed either way.
- A Land that tries to start while the waiter holds its hub's land lock waits on it, under the hub pause, until the waiter ends.

## Acceptance criteria

- `lyx hibernate request` with no flag, on Linux, from the orch or an external caller, takes every discovered hub's land lock, pauses each hub, confirms, runs the hibernate seam once, and after a detected sleep resumes only the entries it created, releases the locks and records `done`.
- A land lock held by a Land in flight delays the pause until the Land releases it.
- `--after <slug>` does not fire while the slug's run is stepping, fires once it reaches its terminal or its status file is gone, and on a halt or failure stays `waiting` and queues one orch notice.
- `--all` fires only once every batten run stepping at request time has landed.
- `--after` naming a slug with no stepping run, `--all` with none stepping, both flags together, and either flag under a live hub pause are refused with a way forward.
- A hub already holding a live hub entry gets no new entry and is not resumed by the waiter.
- A hub found by discovery during confirmation is paused and confirmed before the hibernate.
- Confirmation that does not pass within the bound, whether held by a busy session or a network operation in flight, records `failed`, resumes the waiter's entries, queues one orch notice, and never runs the hibernate seam.
- A Pause error on one hub records `failed` and resumes the entries already created.
- A non-zero hibernate exit, and a zero exit with no sleep seen within the bound, each record `failed` and resume the waiter's entries.
- `lyx hibernate cancel` during `waiting` ends the request with no pause added; during `confirming` it resumes the waiter's entries and the hibernate seam is never run; during `hibernating` with a live waiter it is refused.
- `cancel` with a dead waiter in `confirming` or `hibernating` resumes every hub the record names except one marked already paused, including a hub recorded before its Pause returned.
- An operator `lyx resume` lifting a waiter-created entry during `pausing` or `confirming` records `cancelled` and resumes the waiter's other entries.
- A second `request` while one is non-terminal is refused naming `lyx hibernate cancel`; a `waiting` request with a dead waiter gets a new waiter; two concurrent `request` calls yield one request.
- The hidden verb run with no record, a terminal record, or a different token changes nothing.
- `request` and `cancel` from an agent are refused with the ask-your-parent way forward; `status` is allowed for any caller.
- `request` refuses on a non-Linux GOOS and on WSL.
- Hub discovery includes the requesting hub and each hub resolved from a lyx process's cwd, and skips a cwd outside any hub and a hub under the lyx temp root.
- `lyx p` and `lyx r` expand to `lyx tender pause` and `lyx tender resume`.
- The `PATTERN-hibernate-single-site` scan fails on a `systemctl` sleep verb outside the hibernate engine.

## Notes for the plan writer

- tendercli's `callerOf` and `refuseCaller` are the caller-rule precedent for D2.
- `tenderengine.PauseReport`'s already-paused field tells D6 whether the waiter created the entry.
- batten's land lock is `LandLock` in `internal/battenshed`, a non-blocking try-acquire with no holder record.
- The orch notice goes through whichever single orch-reaching path exists at Plan time, `orchengine.QueueNotice` today and creel's post if `creel` has landed; the waiter spawn likewise goes through whatever launch path `spawn-audit-log` has left.
