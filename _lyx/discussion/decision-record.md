# Decision record: dobby-overview-window

## Goal

Show what a worktree's dobby does and whether it lives, in the overview window that worktree's reed session already has, and once on demand.
An operator looking at the overview of the prime or a beam sees that worktree's dobby alive or not, the runs it drives with their row and last pass, and each released task that waits with what it waits on.

## Scope

- The overview pane, `lyx batten overview`, adds a dobby section for the worktree it runs in when that worktree is the prime or a beam; no new window or strand.
- A new operator verb `lyx beam status` prints the same view once, for the prime's dobby and the dobby of every local beam.
- Dobby writes one log line to its log each time it starts a task and each time it gives a run a pass.
- Out of scope: any change to what dobby decides, to its cadence, or to the overview's run lines and orch lines.

## Decisions

### The view is built by one pure function in dobbyengine

`dobbyengine.BuildView` takes the liveness, the state record, the beam config, the run listing, the board listing and the pause coverage, and returns a `View`: alive, `max_runs`, the runs driven and the waiting tasks.
The waiting reasons come from the same predicate `readyToStart` uses, so the view can never explain a wait the scheduler does not have.
Rationale: the scheduler and the view must agree, and a pure function is testable without a hub.

### A task waits for: open dependencies, another machine's ownership, a covering pause, a refused start's backoff, and `max_runs`

A released task is a board task with an empty status whose parent is this dobby's worktree.
Each waiting task lists every reason that applies: the open dependencies by slug, owned elsewhere, paused, the last refusal text with the next try, and `max_runs` reached.
A released task with no reason shows as ready, which dobby starts on its next tick.
Rationale: these are exactly the conditions `startReady` checks; listing all of them shows the full way forward at once.

### The refusal text is kept in the start record

`StartState` gains the last refusal text, written by the refused start and cleared with the rest of the record.
Rationale: the record held the backoff and the noticed texts only, so the reason of a refused start was not readable once the notice went out.
The record is tolerant JSON, so an older binary keeps the key.

### The board read resolves the parent and owner of every released task

`TaskView` carries the open dependencies by slug instead of a done flag, and its parent and owner are read for every task with an empty status, not only those whose dependencies are done.
Rationale: a task waiting on dependencies must be placed under its dobby to be shown.
Bound: one more cached `hubgeom.ParentOf` per released task per process; the answer is cached per slug, as before.

### The hub reads move from beamcli to battencli

The run and task listings dobby reads over the hub move from `internal/beamcli` to exported functions in `internal/battencli`, which both the dobby wiring and the overview pane call.
Rationale: `beamcli` imports `battencli`, so the overview in `battencli` cannot reach the reads where they are; the runs read are batten runs, and `battencli` already exports the other wirings dobby reuses.

### The view renders once, in battencli, for both the pane and the verb

A rendering in `battencli` turns a `View` into lines; the pane appends them below the run lines, after a blank line, and `lyx beam status` prints them beside the structured view in its envelope.
Times still growing render in minutes, so the pane's text changes at most once a minute and its poll keeps backing off.

### Dobby's log line goes through a told seam

`Deps` gains a `Log` seam taking one line; the hub wiring writes it to the dobby process's standard output, which `SpawnDetached` points at `dobby.log`.
The line carries the time, `start` or `pass`, the slug and, for a pass, its kind.
A nil seam logs nothing.

## Constraints

- PATTERN-cwd-resolution: dobbyengine stays told-geometry and imports no lyxcwd or CLI package; its import allowlist test lists every import.
- PATTERN-tolerant-records: the new start-record key is optional and written through `UpdateState`.
- PATTERN-daemon-wakeups: the pane's poll keeps its backoff; the dobby section adds no wake.
- PATTERN-cli-cobra: `lyx beam status` is a subcommand of the existing beam tree.
- PATTERN-pair-parent: a task's parent is read only through `hubgeom.ParentOf`.
- PATTERN-test-speed: tests are tier0 where no hub is needed.

## Auto-mode assumptions

- "Each worktree's overview" is the pane of `lyx batten overview` that `lyx orch start` ensures in the prime and in each beam; a pane in any other worktree shows no dobby section.
- `lyx beam status` shows every local dobby, as `lyx beam start` considers them, rather than only the acting worktree's.
- "Last pass" is the time the run's last pass exited, from the state record, and a held driver lock shows as a pass in flight.
- A first pass in flight that has left no status yet shows its task as ready, since its in-memory flight is not in any file.

## Open risks

- The pane now reads the board, the owners and beam.yaml on every poll; on a large board that adds file reads per second at most, with no git call once each parent is cached.
- A parent cached in a long-lived pane can go stale if a released task's project changes before its pair exists; dobby itself has the same cache.

<!-- lyx:carry-over Code-Review -->
- Code-Review, round 1: closed by an accepted circling decision; a finding below was addressed by the round's fixer or is still unfixed.
  Review: _lyx/reviews/code/round-1-review.md; fixer report: _lyx/reviews/code/round-1-fixer-report.md.
  - dobbytasks-change-untested (class design, severity MEDIUM, open in rounds 1)
  - overview-dobby-section-in-task-pair (class design, severity BLOCKING, open in rounds 1)
  - overview-poll-takes-dobby-and-driver-locks (class design, severity MEDIUM, open in rounds 1)
  - refusal-clearing-unasserted (class design, severity MEDIUM, open in rounds 1)
<!-- /lyx:carry-over Code-Review -->

## Acceptance criteria

- A dobby whose lock is held shows alive; a free lock shows not running with `lyx beam start` as the way forward.
- A running run shows its state, row and the minutes since its last pass exit; a held driver lock shows a pass in flight; a done run is not shown.
- A released task with open dependencies lists them; one another machine owns, one a pause covers, one with a refused start's text and next try, and one held back by `max_runs` each show that reason; a task with none shows as ready.
- A task of another worktree, a draft task and a note are not listed.
- The refused start's text is recorded in the start record and cleared when a pass claims the task or its entry changes.
- Dobby's log receives one line per task start and one per pass spawn, naming the slug and the pass kind.
- The overview frame of a prime or beam worktree carries the dobby lines below the run lines; a task pair's frame carries none.
- `lyx beam status` prints the view of the prime's dobby and of each local beam's in its envelope.

## Notes for the plan writer

- `readyToStart` and the counting in `runsInFlight` are the logic the view must share; factor rather than restate them.
