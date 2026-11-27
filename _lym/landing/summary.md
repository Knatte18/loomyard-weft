# dobby: mail the parent on run stops and on gate slots held too long


The dobby of a run's parent worktree now watches every run it drives and mails the nearest live orch,
so the orch needs no Monitor kept by hand, and the watch outlives the orch's context cycles.
Existing notices are unchanged; the new lines are added to them.

## Run watch

On each scheduling tick the dobby reads every listed run: its derived state, batten's state and row,
the child's state and row, and an episode stamp that tells a second stop at the same row from the first.
It mails changes as `[run watch] <slug>: <derived state>; batten … ; child … ; since <time>`, with the reason when one is set.

The new per-task key `beam.run_watch` sets the level:

- `stops` (default): one line per stop. A stop is any reading whose state is not running, a child's shed done included.
- `rows`: one line per change, row moves and resumes included.
- `off`: no watch line.

## Gate-hold notice

On a probe tick the dobby reads the hub's gate slot holders.
A hold by a driven run's task worktree that reaches its bound in awake time gets one `[dobby notice]`,
naming the slots, tier, site, start, bound, the holder's process tree and the marked processes outside it.
The dobby only reports; it kills nothing and frees no slot.

The bound is `beam.gate_hold_factor` (default 1.5) times the tier's `gate.tiers.<tier>.timeout_min`.
A holder with no tier or timeout gets `beam.gate_hold_min` (default 90) minutes. A factor of 0 turns the notice off.

A tier run now records its environment marker in the slot holder record,
which is how a process that left the holder's tree is still found.
An older record reads as having no marker.

## Design decisions

- **The default is on.** Every hub gets the watch without setup, and each stop reaches the orch once more, ahead of batten's notice.
  An upgrade sends one line at once for every run already standing at a stop.
- **Watch lines carry no creel topic or run tag,** so none replaces another unread one, and teardown drops none.
- **Owed lines live in the dobby state record** and post oldest first.
  They wait while no orch strand is recorded.
  A line is dropped with a Warn after three failed posts or an hour of awake time.
- **The done-run clear now waits** until no watch line is owed and the run's `beam.run_watch` has been read.
- **A landed run covered by `beam.land_notice` sends no watch line,** since the landing notice reports it.
  The level is not re-read past Land, where the task's worktree is gone.
- **The episode stamp is history length plus the last entry's folded repeats,**
  since a repeated block can fold into the previous history entry.
- **A child written after its run was listed is read on the next tick,** so one stop is not told twice.
- **The gate-hold line names at most 20 descendants and 20 marked processes,** then counts the rest.
- **The marker is the nested-gate environment variable.**
  A `lym gate test` nested inside a slot replaces it, so that inner run's processes are not found through the outer marker.

## Limits

- A cancel that removes the run's records posts no watch line; only the cancel's own notice, tried once, reports it.
- The watch runs only on ticks whose scheduling steps run, so an anchor move, an adopt hold or an owed catch-up delays it.
- Slots held outside a driven run, and runs no dobby drives, are not watched.
- On Windows the gate-hold notice reports the process tree as unreadable.
- A suspend the dobby did not record counts as awake time and can bring a gate-hold notice early.

## To check by hand

Each of these test gaps has tests in the diff; confirm that they close it.

- Selection of a blocked run's reason (batten's as the fallback, the child's taking precedence)
  had no test in `internal/dobbyengine/runwatch_test.go`.
- A failed watch record holding the done-run clear had no test in `internal/dobbyengine/donerun_test.go`.
  The test for it skips on Windows and as root.
- The hub wiring of the gate-hold notice was pinned only for tier and timeout
  in `TestDobbyGateHolders`, `internal/beamcli/dobbydeps_integration_test.go`.
- A holder working in the task worktree itself had no test in `internal/dobbyengine/gatehold_test.go`.
- Detection of a second stop through batten's episode and the child's history length
  had no test in `internal/dobbyengine/runwatch_test.go`.

## Merge danger

**Door:** one-way
**Blast radius:** dobby

The door is one-way because the dobby sends mail, which a revert does not recall.
