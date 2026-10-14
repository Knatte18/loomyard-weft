# Decision record — hibernate-trigger

## Goal

Let the operator, from the phone through the orch, queue a hibernate of the machine that lyx's Go code carries out on its own once every session in the hub is safely paused.
No LLM session runs the hibernate, so no session is left hanging mid-call when the machine goes down.
Add the short `p` and `r` shortcuts beside `lyx pause` and `lyx resume`.

## Scope

In:

- A new `lyx hibernate` CLI module with `request`, `cancel` and `status` verbs, and a hidden detached daemon verb that `request` spawns.
- A hibernate request file per hub, recording the trigger, the requester, the daemon, the pause entry it owns and the outcome.
- Three triggers: now, after one slug's landing, and after every batten run in the hub is out of flight.
- The firing sequence: a hub-wide tender pause, a confirmation check that includes the orch, `systemctl hibernate`, wake detection and the lift of the daemon's own pause entry.
- A lift of one pause entry by its identity, added to the pause engine for the daemon and `cancel`.
- The orch's command index and role stencil name `lyx hibernate` as the way to hibernate.
- Shortcuts `p: [tender, pause]` and `r: [tender, resume]` in `internal/shortcutcli/shortcuts.yaml`.
- An addition the board entry does not name: a standing deny of the machine sleep commands in the settings lyx writes for every session it spawns (Decision 10).
- Docs: the new package's `doc.go`, the module table in `docs/overview.md`, and `internal/shortcutcli` docs if they list shortcuts.

Out:

- Any platform but Linux: on others `request` and `cancel` refuse as unsupported.
- Suspend, poweroff and reboot as lyx verbs.
- Pausing sessions of other hubs on the machine (see Open risks).
- Changes to batten's recipe: no new row, and Land is untouched.

## Decisions

1. **The trigger lives in a new module, `hibernate`, with its own engine and CLI packages.**
   The engine drives tender's pause engine through seams, as `internal/tendercli` wires it today, and is told every path (`PATTERN-told-geometry`).
   The CLI is a cobra subtree under the root in `cmd/lyx/main.go` (`PATTERN-cli-cobra`).
   It is hub-only and not standalone-capable, so it resolves the hub through `lyxcwd` from any worktree, as tendercli does, and `internal/cliwire` does not serve it (`PATTERN-cliwire-sole-wiring` does not apply).

2. **Verbs.**
   - `lyx hibernate request` queues a hibernate to fire as soon as the pause is confirmed.
   - `lyx hibernate request --after <slug>` fires once that slug's batten run has finished its Land row: its status reads done, or its current row is past Land.
   - `lyx hibernate request --after-all` fires once no batten run in the hub reads running; a halted or awaiting run counts as out of flight, since it waits on the operator, and with no batten run at all it fires at once.
   - `lyx hibernate cancel` withdraws a request and lifts the pause entry it owns (Decision 8).
   - `lyx hibernate status` reports the request, its trigger, its phase, its reason, the pause entry it owns and the daemon's liveness, and writes nothing.
   `--after` and `--after-all` are exclusive.
   `request` refuses a `--after` slug with no batten status, listing the slugs that have one; a slug whose run has already finished Land fires at once.
   `request` returns once the daemon is spawned, so the orch's turn can end before anything is paused.

3. **Callers.**
   The caller is read from the strand-name variable reed injects, parsed through `internal/agentname` (`PATTERN-agent-name`), with tendercli's rule: empty is external, the orch's name is the orch, any other name is an agent.
   The orch and an external caller may run every verb; an agent of any pair is refused on `request` and `cancel`, with the way forward of asking its parent session; `status` takes no caller check.
   Bound: the check guards against a mistaken call, as tender's does, and the agents' settings deny clearing the variable.

4. **One request per hub, in an ephemeral file under the hub's `.lyx`.**
   The request file sits beside the pause record, its path declared through `internal/lyxdirs` (`PATTERN-lyxdirs-single-declarer`, `PATTERN-durable-vs-ephemeral-state`), written through `internal/state` and read tolerantly (`PATTERN-tolerant-records`).
   It holds the trigger, its slug, the requester, the time, the daemon's pid and start time, the owned pause entry (Decision 7), the phase (`waiting`, `pausing`, `confirming`, `hibernating`, `resuming`, then a terminal `done`, `failed` or `cancelled`) and the last reason.
   Every write runs under the request file's lock: `request` checks, writes and records the spawned daemon's pid in one hold, so two racing requests yield one daemon.
   A `request` while a request with a live daemon is non-terminal is refused, naming `lyx hibernate cancel`.
   A `request` over a terminal request replaces it; over a dead daemon's non-terminal request it first lifts that request's owned entry, as `cancel` does, then replaces it.
   This departs from the board's "flag on the machine": the request is hub-scoped, since lyx keeps no registry of a machine's hubs, the pause record is hub-scoped, and the operator runs one hub.

5. **A detached daemon, not batten, fires the hibernate.**
   `request` spawns the hidden `lyx hibernate watch` as a detached process, by the same self-spawn and binary resolution the orch watcher uses, and logs the spawn (`PATTERN-spawn-observability`).
   The daemon polls its trigger at most once a second and backs off while it waits (`PATTERN-daemon-wakeups`), logging per `internal/logger`'s level policy.
   It reads batten statuses through paths the CLI tells it, taken from their owning declaration (`PATTERN-shed-run-directory`).
   A slug whose batten run halts before Land leaves the request waiting, and `status` says why; the operator cancels it.
   A status file that goes missing while the daemon waits ends the request `failed` with the reason.

6. **Firing: pause, confirm, hibernate.**
   When the trigger holds, the daemon runs tender's pause engine at hub scope with the caller label `hibernate`, so the pause covers every pair's sessions and leaves the orch to its live watcher, as `lyx pause` does.
   The pause is confirmed when all of these hold at one reading:
   - no pair session reads busy, `still_busy` or refused, and none is listed with background work;
   - the orch's session, read through the prime's state, reads idle;
   - no covered network operation is in flight;
   - no batten land lock is held, so no landing is in flight.
   A held parent lock does not block confirmation, since a paused run keeps its parent lock by design.
   An unconfirmed pause is re-read from tender's status session states, network operations and the land locks every five seconds, at most 120 times, ten minutes in all.
   Right before the hibernate call the daemon checks that the request is not cancelled and that its owned entry, when it has one, is still live; a lifted entry ends the request `cancelled`, with no hibernate.
   Once confirmed, the daemon runs `systemctl hibernate`, logging the spawn and its exit, and moves to `hibernating`.
   Bound: the daemon sends no key and no text of its own; every interrupt is the pause engine's, under its existing guards.
   It never runs the hibernate without a confirmed pause, and 120 re-reads without confirmation end the request `failed`.

7. **The daemon owns at most one pause entry, by identity, and lifts it on wake.**
   The pause engine gains a lift of one entry by scope, caller and paused-at time, which lifts nothing when the live entry of that scope differs.
   When the daemon's pause adds an entry, the request file records its scope, caller label and paused-at time; when a hub entry was already live, the pause adds none, the request records no owned entry, and the daemon and `cancel` lift nothing.
   `systemctl hibernate` returns once the job is queued, so the daemon detects the wake through `pauserecord.SleepWatch` over the monotonic awake clock, observed each poll across the call.
   On a sleep seen after the call, it lifts its owned entry, moves to `done`, and queues one orch notice through `orchengine`'s notice queue saying the machine woke and whether a pause was lifted (`PATTERN-orch-pane-single-writer`).
   An owned entry already lifted by then is not an error.
   With no sleep seen within five minutes of the call, or on its non-zero exit, it lifts its owned entry the same way, ends `failed` with the reason, and queues an orch notice naming it.
   Rationale: the pause exists only to make the hibernate safe, and the operator already hibernates with runs live, so runs carry on after the wake with no manual `lyx resume`.

8. **Cancel.**
   `cancel` marks the request `cancelled` under the request file's lock, stops a live daemon, and lifts the request's owned entry.
   `cancel` of a request with a live daemon in `hibernating` refuses, since the machine is going down; a dead daemon's request is cancelled in any phase, which is the way out after a crash or a failed resume from hibernation leaves the hub paused.
   A cancel with no request, or a terminal one, succeeds with nothing cancelled.
   Bound: a cancel that lands while the pause engine is mid-sequence relies on its existing rule that an entry no longer live stops further keys; a cancel after the daemon's last check before the hibernate call misses by one file read.

9. **Platform.**
   On Linux the daemon runs `systemctl hibernate`; `request` first checks that `systemctl` is on `PATH`.
   On any other platform `request`, `cancel` and the daemon refuse as unsupported; `status` still reads.

10. **Spawned sessions are denied the machine sleep commands.**
    The standing deny table in `internal/shuttleengine/claudeengine/settings.go` gains a Bash deny for `systemctl` and `loginctl` with the verbs `hibernate`, `suspend`, `hybrid-sleep` and `suspend-then-hibernate`, its notice line built from the same table.
    Its steer says a hibernate goes through `lyx hibernate request`, which a pair agent asks its parent session to run.
    Bound: it refuses only those two commands with those four verbs; `systemctl status` and every other verb pass.
    Rationale: the orch hibernated the machine ad hoc on 2026-10-13, and the board's direction is that no LLM session runs the hibernate.

11. **The orch knows the verb.**
    `lyx hibernate` appears in the operator command index the orch's role file carries, and the orch role stencil says a hibernate request from the operator goes through `lyx hibernate request`, with `--after <slug>` or `--after-all` when the operator names a task or the runs.

12. **Shortcuts `p` and `r`.**
    `internal/shortcutcli/shortcuts.yaml` gains `p: [tender, pause]` and `r: [tender, resume]`, beside `a`.
    The board entry reserves this call for the operator; it is an auto-pick of the board's proposed names, for parent review to confirm.

13. **Sandbox coverage.**
    The `hibernate` module is excluded from the sandbox suite with the reason that its daemon hibernates the host (`PATTERN-sandbox-coverage`).

## Constraints

- `PATTERN-guarded-interrupt`, `PATTERN-pane-text-injection`: the daemon sends nothing to a pane itself.
- `PATTERN-no-denied-recovery`: no refusal or stencil names a denied command; the sleep deny's steer names `lyx hibernate request`, which is not denied.
- `PATTERN-shuttle-provider-seam`: the deny lives under `claudeengine`, as the other standing denies do.
- `PATTERN-test-speed`, `PATTERN-test-economy`: engine tests drive the sequence over fake seams and an injected clock; no test runs `systemctl`.
- Test budget: a package past its row in `cmd/lyx/testdata/test-budget.yaml` lists that file under `Edit:`.

## Auto-mode assumptions

- "Every instance" means every session of the hub (Decision 4).
- The daemon lifts its own pause on wake (Decision 7).
- `--after-all` treats halted and awaiting runs as out of flight (Decision 2).
- The confirmation window of 120 five-second re-reads and the five-minute wake window are fixed in code, with no config key.
- The deny of the sleep commands (Decision 10) and the `p` and `r` shortcuts (Decision 12).

## Open risks

- A second hub on the same machine is not paused before the hibernate; its sessions would be cut mid-call.
- The operator's own typing into a session between confirmation and the hibernate call is outside lyx's control.
- `--after <slug>` fires after Land's deploy, so the daemon runs from a replaced binary and `lyx tender status` lists it among the processes running one.
- The sleep deny is Claude-engine settings; a second engine (`codex-engine`, `llm-agnostic-engine`) needs its own.
- `systemctl hibernate` may need polkit permission on another machine; on the operator's machine the orch ran it on 2026-10-13.
- A failed resume from hibernation boots cold with the daemon gone and its pause entry live, so the hub stays paused until `lyx hibernate cancel`.

## Acceptance criteria

- `lyx hibernate request` with no trigger spawns the daemon and returns at once; the daemon pauses the hub, confirms, runs the hibernate command, detects the wake, lifts only its own entry and ends `done` with one orch notice queued.
- `--after <slug>` waits while the slug's run is before Land, and fires once Land is finished, including when teardown then halts; a slug already past Land fires at once; a slug with no batten status is refused at `request`.
- `--after-all` fires once no batten run reads running, and at once with no batten run.
- Confirmation is withheld while a pair session is busy, `still_busy`, refused or listed with background work, while the orch reads busy, while a network operation is in flight, or while a land lock is held; a held parent lock alone does not withhold it.
- 120 re-reads without confirmation end `failed`, lift the owned entry and notify the orch, with no hibernate command run.
- A non-zero hibernate command exit, and a wake not seen within five minutes, each lift the owned entry, end `failed` and notify the orch.
- An operator `lyx resume` after the pause and before the hibernate call ends the request `cancelled` with no hibernate.
- A request made while a hub pause is live owns no entry, and neither the daemon nor `cancel` lifts that pause.
- `cancel` before firing stops the daemon and leaves no owned entry; after the pause and before the hibernate call it stops the hibernate and lifts the entry; with a live daemon in `hibernating` it is refused; with a dead daemon it cancels in any phase and lifts the owned entry; with nothing pending it succeeds with nothing cancelled.
- A second `request` while one with a live daemon is non-terminal is refused naming `lyx hibernate cancel`; two racing requests spawn one daemon; a `request` over a dead daemon's request lifts its owned entry and replaces it.
- `status` reports trigger, phase, reason, owned entry and a dead daemon, and writes nothing.
- An agent of a pair is refused on `request` and `cancel`; the orch and an external caller are not.
- On a non-Linux platform `request` and `cancel` refuse as unsupported and `status` reads.
- A spawned session's Bash call of `systemctl hibernate`, `systemctl suspend` or `loginctl hibernate` is denied with the steer; `systemctl status` passes; the deny notice lists the new deny.
- `lyx p` runs `lyx tender pause` and `lyx r` runs `lyx tender resume`.

## Notes for the plan writer

- The tender sequences live in `internal/tenderengine`, wired by `internal/tendercli/deps.go`; `ResumeRequest` lifts by slug today, and `pauserecord.Entry` carries scope, caller and paused-at time.
- The orch watcher's detached spawn in `internal/orchcli` is a model for the daemon spawn; `internal/orchengine/notice.go` holds the notice queue.
- `pauserecord.SleepWatch` and `MonotonicAwake` already detect sleeps; `internal/battenshed` uses them for its idle clocks.
- Batten's land lock and status paths are in `internal/battencli/paths.go`.
- The standing denies and their notice share one table in `internal/shuttleengine/claudeengine/settings.go`.
- `cmd/lyx/shortcuts_test.go` pins the shortcut file.
