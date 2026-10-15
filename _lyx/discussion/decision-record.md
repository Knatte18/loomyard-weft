## Goal

The operator, usually from the phone through the orch, can ask lyx to hibernate the machine now or once named runs have settled.
No LLM session runs the hibernate.
The orch records a request, and lyx's Go code pauses the hub, confirms the pause, runs the hibernate and, on wake, lifts the pause it added.
Beside that, `lyx p` and `lyx r` become short forms of `lyx pause` and `lyx resume`.

## Scope

In:

- A new module `lyx hibernate` with the operator verbs `request`, `cancel` and `status`, and a hidden daemon verb `watch`.
- Triggers: now, after named batten runs settle, and after every batten run in flight at request time settles.
- The fire sequence: land-lock wait, hub-wide tender pause, confirmation, `systemctl hibernate`, wake detection, lifting the pause the request added, and an orch notice.
- A narrow `tenderengine` addition that resumes one named hub entry, since an empty-slug resume lifts every live entry.
- A standing deny of the machine sleep commands for every lyx-spawned session, the orch included, steering to `lyx hibernate request` (D13).
- One line in the orch role stencil pointing a hibernate request at `lyx hibernate request`.
- The shortcuts `p` and `r` (D11).
- Docs, the PATTERN entry and the sandbox exclusion (D12).

Out:

- Other hubs on the same machine (D10).
- Stopping or holding background forks, background shells and Monitors during a pause, which stays with the `pause-background-work` board entry.
  This task decides only that background work does not withhold the hibernate's confirmation (D5).
- Platforms other than Linux (D9).
- A config module for the hibernate's timings. Its bounds are constants in the engine.

## Decisions

### D1 — A `hibernate` module: `request`, `cancel`, `status` and a hidden `watch`

Decision: Add `internal/hibernatecli` (a cobra subtree under the root in `cmd/lyx/main.go`) and `internal/hibernateengine` (the sequences over seams, told every path and every clock).
The verbs run from the hub's prime worktree only, as `lyx orch` and `lyx batten` do, and refuse from a pair and from `_board`.
`request`, `cancel` and `status` carry the operator audience annotation, so the orch's command index lists them.
Rationale: the orch runs from the prime, and the pause is hub-wide, so a pair has no reason to request a hibernate.
Bound: from a pair, every verb refuses.
In the prime, any session may request, and D13's deny leaves this module's confirmed sequence as an agent's only way to hibernate.
`watch` is hidden, and it refuses unless the request record names the request id it was spawned with and it can take the request's lock, so running it by hand does nothing that `request` did not record.

### D2 — `request` records the trigger and spawns the watcher, then returns

Decision: `lyx hibernate request` takes zero or more `--after <slug>` flags, or `--all`, but not both.
With neither, the request fires at once.
`--after` names batten runs, and a slug named twice counts once.
`--all` takes every batten run in this hub that has not settled at request time. A run started after the request is not waited on.
Each `--after` slug must have a batten status in this hub, or `request` refuses and names the slugs that have one.
A named run that has already settled counts as settled, so a request whose runs have all settled fires at once.
Under the request's lock, `request` refuses while the recorded watcher is live, naming `lyx hibernate cancel`.
Otherwise it writes the request record with a fresh request id under the prime's `.lyx` (path declared through `internal/lyxdirs`, written through `internal/state`), spawns `lyx hibernate watch` detached with that id, and returns at once with the record and the watcher's pid on the envelope.
The spawn is logged through `internal/logger`, as the orch watcher's spawn in `internal/orchcli/spawn.go` is, and sits behind an injected spawner so tests never re-exec the binary.
Rationale: the orch's turn ends right after the call, so no LLM session is mid-call when the machine goes down.
Bound: one live request per hub, with `request` and `watch` serialised on the request's lock.
A record whose watcher is dead is replaced, after `request` first lifts the pause entry that record says it added (D6).

### D3 — `watch` is the only code that hibernates

Decision: The watcher holds the request's lock except while `cancel` or `status` reads the record.
It runs, in order: wait for the trigger (D4), wait for the land lock and pause and confirm (D5), hibernate (D6), and after wake lift its pause and notify (D7).
It removes the record on every path that ends it.
This departs from the board entry's "who fires it: batten after Land": the watcher fires instead, once each named run's batten is done at `Worktree-Teardown`, which is after Land and its deploy.
A new test-guarded `PATTERN.md` entry under "Daemons and waits" pins it: running the machine's hibernate is done only by `internal/hibernateengine`'s watcher, after a confirmed hub pause, and no other production file names `systemctl hibernate`.
Rationale: one site that runs the command keeps the "pause confirmed first" rule checkable, and a watcher can serve "now" and several slugs, which a Land row cannot.

### D4 — When a named run has settled

Decision: The watcher reads each named slug's batten status, and its child run's status as batten's Run-Shed wait reads it, through the existing readers, never by spawning `lyx`.
A slug has settled when one of these holds:

- its batten run is done at `Worktree-Teardown`, so Land and its deploy have finished;
- its batten run is `blocked` or `failed`;
- its child run is `blocked` or `failed`, which leaves batten itself `running` in Run-Shed's halted-child wait.

A run that is `running`, `paused` or `awaiting`, at either level, has not settled, since the orch, the operator or the run may still act on it.
A run paused before the request therefore holds the request until it is resumed and settles, or until `cancel`.
Counting a halted run widens the board entry's "when `<slug>` lands". D7's notice names how each slug settled, so a slug that did not land is visible on wake.
The wait's poll file joins the declared unbounded-wait list `cmd/lyx/daemonwakeups_test.go` guards, with a 1-second base backing off to 30 seconds while nothing changes.
Rationale: a halted run makes no progress without the operator, so waiting on it would keep the machine up for nothing.
Bound: the trigger wait has no time limit. It ends when every run settles, or on `cancel`.

### D5 — Wait for the land lock, pause the hub and confirm

Decision: Once triggered, the watcher first waits until no batten run holds the hub's land lock (battenshed's land lock, read through its own seam), for at most 35 minutes, longer than Land's 30-minute deploy timeout.
It then runs `tenderengine.Pause` for the whole hub in-process with tender's default grace, through `Deps` built the same way `tendercli` builds them.
The pause entry's caller is `EXTERNAL`, since the watcher has no strand.
The request record keeps the entry's identity and whether this pause added it or found a hub entry already live.
The watcher then polls, at the same reading, until all of these hold:

- no session reads busy with cause `turn`, the orch included;
- no network operation is in flight;
- no batten run holds the land lock.

A session busy only with cause `background` does not withhold, and neither does a held parent lock, since a run holds one from Merge-In through Finalize, including the whole PR-Gate wait.
The orch is interrupted by its own watcher at the grace end. With no orch watcher live, an orch in a running turn withholds until the bound.
Confirmation is bounded at 10 minutes from the pause.
When the land-lock wait or the confirmation bound passes, the watcher lifts the pause it added (D6's rule), queues an orch notice naming each reading that withheld, and ends without hibernating.
Rationale: the operator already hibernates with runs live, so the bar is that no turn, no network operation and no landing is mid-flight. A background call dropped across a hibernate stalls on an API error, which `lyx tender revive` recovers.
Bound: confirmation lets through background work, which may lose an in-flight API call across the hibernate. It never lets through a running turn, a network operation or a held land lock.

### D6 — Run `systemctl hibernate` and detect the wake

Decision: Right before the command, under the request's lock, the watcher re-reads that the request is not cancelled and that its hub entry is still live.
If either fails, it ends without hibernating, lifts what it added, and queues a notice.
It then records the phase `hibernating` and runs `systemctl hibernate` through a told command seam, logging the spawn.
A non-zero exit lifts the pause the watcher added, queues an orch notice with the command's stderr, and ends.
Wake detection uses `pauserecord.SleepWatch` and its `SleepTolerance`, starting from the command's start, so it holds whether `systemctl` returns before the machine goes down or only after the wake.
If no sleep has ended within 2 minutes of a zero exit, the watcher treats the hibernate as not having happened and handles it as a non-zero exit.
Lifting the pause: when the watcher's pause added the hub entry, it resumes that entry only, through the `tenderengine` addition in Scope.
When a hub entry was already live before the request, the watcher leaves it in place.
Rationale: a told command and a told awake clock keep the sequence testable with no real hibernate. Reusing `SleepWatch` keeps one definition of a machine sleep.
Bound: the watcher lifts only an entry it added, so a request never resumes an operator's own pause.

### D7 — After wake, lift the pause and tell the orch

Decision: After a wake the watcher lifts its pause (D6's rule) and queues one orch notice through `orchengine.QueueNotice`.
The notice names when the request was made, how each slug settled, and whether the hub was resumed or left paused.
It also names each session that was busy with background work at confirmation, with `lyx tender revive` for any that stalls.
It names each batten run blocked at Land since the pause, with its way back: `lyx batten goto <slug> --to Land`, then `lyx batten run <slug>`.
Every failure path in D5 and D6 queues one notice the same way.
The orch watcher delivers no notice while a hub entry is live, so under a pause the watcher left in place, the notice arrives at the operator's resume.
Rationale: the orch learns what happened from its own queue, and the orch watcher stays the only writer into its pane (`PATTERN-orch-pane-single-writer`).

### D8 — `cancel` and `status`

Decision: `lyx hibernate cancel` withdraws the request under the request's lock.
With a live watcher, it marks the record cancelled, and the watcher lifts any pause it added and ends at its next check.
Before the watcher records `hibernating`, cancel always wins.
After that, `cancel` reports that the hibernate is under way and changes nothing.
With a dead watcher, `cancel` lifts the pause entry the stale record says it added and removes the record.
`lyx hibernate status` writes nothing. It reports the record (trigger, slugs and how far each has settled, phase, the pause entry and whether the request added it), whether the watcher is live, and during confirmation what withholds.
Rationale: from the phone the operator needs to see and undo a queued request through the orch.
Bound: `cancel` touches only the hibernate record and the pause entry that record added.

### D9 — Linux only

Decision: On a platform other than Linux, `request` refuses with "hibernate is unsupported on <GOOS>", and its way forward is that the operator hibernates the machine themselves.
`status` and `cancel` work everywhere.
Rationale: the operator runs lyx on Linux only, and an agent cannot take a way forward its settings deny (`PATTERN-no-denied-recovery`).

### D10 — The request covers its own hub, not every hub on the machine

Decision: The request and its pause cover the hub the request was made in.
This departs from the board entry's "a hibernate flag on the machine" and "a pause on every instance": the flag lives in the hub's prime, and "every instance" is read as every session of that hub.
Rationale: lyx has no registry of a machine's hubs, and the operator runs one hub per machine.

### D11 — Shortcuts `p` and `r`

Decision: Add `p: [tender, pause]` and `r: [tender, resume]` to `internal/shortcutcli/shortcuts.yaml`, after `pause` and `resume`, and name them on the shortcuts line in `docs/overview.md`.
Rationale: they sit beside `pause` and `resume` the way `a` sits beside `attach`. Neither name is taken.

### D12 — Docs and sandbox coverage

Decision: The module's design goes into `internal/hibernateengine/doc.go` and `internal/hibernatecli/doc.go`.
`docs/overview.md` gets the module's table rows and a module entry, and `PATTERN.md` gets D3's entry with its guard.
The sandbox suite excludes the module with the reason that it would hibernate the host, and its tests run the sequence against fakes.
Rationale: CLAUDE.md's "Docs land in the same commit" rule, and `PATTERN-sandbox-coverage`.

### D13 — A standing deny of the machine sleep commands

Decision: Add a row to `claudeengine`'s standing denies, installed for every run, the orch included, with no config key.
It refuses a Bash call that runs `systemctl` with `hibernate`, `suspend`, `hybrid-sleep` or `suspend-then-hibernate`, or runs `loginctl` with the same verbs, and its steer names `lyx hibernate request`.
Its comment states what it lets through, as the other guardrail rows do.
The orch role stencil gets one line saying a hibernate goes through `lyx hibernate request`.
Rationale: the board entry's incident was the orch running `systemctl hibernate` itself. No deny of it exists at HEAD, and the command index alone does not stop it.
Bound: a guardrail, not a barrier. A call behind `bash -c`, a script or another spelling gets through, and the steer and the role line carry the rest.

## Constraints

- `PATTERN-told-geometry`: `hibernateengine` is told every path and its tender, batten-status, land-lock, orch-notice, clock, spawner and command seams, and imports no `internal/lyxcwd`.
- `PATTERN-cli-cobra` and `PATTERN-cwd-resolution`: the CLI is a cobra subtree and resolves the prime through `internal/lyxcwd`, as `tendercli` does.
- `PATTERN-lyxdirs-single-declarer` and `PATTERN-durable-vs-ephemeral-state`: the record and the watcher's log live under `.lyx` in the prime, declared through `internal/lyxdirs`.
- `PATTERN-tolerant-records`: the request record is written through `internal/state` and refuses no unknown key.
- `PATTERN-guarded-interrupt`: the module sends no key itself. Interrupts are `tenderengine.Pause`'s.
- `PATTERN-refusal-way-forward` and `PATTERN-no-denied-recovery`: every refusal names a way forward, and none names a command D13 denies.
- `PATTERN-test-speed`: tests inject the clock and the command, never hibernate the host, and run at the lowest fitting tier.

## Auto-mode assumptions

Every design pick was autonomous, with its reasons in the support log's question ledger. The ones that shape behaviour are D4, D5, D6, D10, D11 and D13.

## Open risks

- A background fork's API call lost across the hibernate leaves its session stalled on an API error, and `orch-split` records that a long-lived driver can stall after a host sleep. D7's notice names `lyx tender revive` for both.
- If the watcher dies after its pause, or the hibernate image fails to restore and the machine boots fresh, the hub stays paused and no notice is queued. The live entry shows in `lyx tender status`, the stale record in `lyx hibernate status`, and `cancel` or a new `request` lifts the entry.
- `systemctl hibernate` may need polkit rights or swap the machine lacks. That surfaces as a non-zero exit and a notice, with the pause lifted.
- The 2-minute no-wake window assumes the machine goes down within 2 minutes of a zero exit. A slower hibernate reads as failed and lifts the pause before the machine goes down.

## Acceptance criteria

- `lyx hibernate request` with no trigger pauses the hub, confirms, runs the hibernate command once, and after a simulated wake resumes the entry it added and queues one orch notice.
- With `--after a --after b`, nothing is paused until both runs have settled. A landed run, a blocked batten run and a blocked child under a running batten all settle. A run awaiting a parent decision, or paused, does not.
- `--all` waits on the runs not settled at request time and ignores a run started later. A request whose named runs have already settled fires at once. A slug passed twice counts once.
- A slug with no batten status is refused at request time, and the refusal names the slugs that have one.
- A second `request` while a watcher is live is refused naming `lyx hibernate cancel`. Two racing requests leave one watcher. A stale record is replaced after its pause entry is lifted. `watch` run without a matching record does nothing.
- The pause waits for a held land lock to be released, and the land-lock wait's bound fails the request without a pause.
- A session busy with cause `turn`, a network operation in flight or a held land lock withholds confirmation. A session busy only with cause `background` and a held parent lock do not, and background sessions are named in the notice.
- When confirmation does not pass within its bound, no hibernate runs, the pause the watcher added is lifted, and the notice names what withheld.
- A cancel, or a lifted hub entry, seen at the re-read right before the command ends the watcher without hibernating.
- A non-zero hibernate exit, and a zero exit with no wake within the window, each lift the pause and queue a notice without retrying.
- A hub entry that was live before the request is left live after wake and after every failure.
- `cancel` before `hibernating` ends the watcher and lifts only its own pause. After it, `cancel` changes nothing and says so. With a dead watcher, `cancel` lifts the recorded entry and removes the record.
- A batten run blocked at Land by the pause is named in the wake notice with its way back.
- `status` writes nothing and reports the phase, how far each slug has settled, and the watcher's liveness.
- From a pair, every verb refuses. On a non-Linux GOOS, `request` refuses.
- The sleep-command deny refuses `systemctl hibernate` and `systemctl suspend`, and lets `systemctl status` through.
- `lyx p` runs `lyx tender pause` and `lyx r` runs `lyx tender resume`.

## Notes for the plan writer

- `tendercli`'s unexported `hubDeps` builds the `tenderengine.Deps` the watcher needs. Share it rather than duplicating it in `hibernatecli`.
- The land lock is battenshed's, with its path in `internal/shedrun`.
- Batten's terminal row is `Worktree-Teardown` (`contracts/recipes/batten-recipe.yaml`).
- New top-level tests may push a package past its row in `cmd/lyx/testdata/test-budget.yaml`. A card that does lists that file under `Edit:`.
- `creel` plans to absorb the orch notice queue, and `spawn-audit-log` plans one recorded launch path for spawns. Use only `orchengine.QueueNotice` and `internal/logger`, so either can take these over.
