# The overview shows its worktree's dobby, and `lyx beam status` prints it

Nothing showed what a dobby does or whether it lives: its log stayed empty unless a tick failed, and a released task could sit unstarted beside a live dobby with nothing saying why (issue #872).
This change adds that view to the existing overview window of the prime and of every beam, prints it once through a new `lyx beam status`, and has dobby log each start and pass.

## What changes

- **The overview pane** (`lyx batten overview`) draws, after a blank line below the run lines, the dobby of the worktree it runs in:
  - a header, `dobby <label>: alive, max_runs N`, or `not running; way forward: lyx beam start`;
  - one line per run the dobby drives that is not done: batten's state, the numbered stage, and `pass in flight` while the driver lock is held, else `last pass Nm ago`, then a full pass owed, an owner elsewhere and a failing count from the state record;
  - one line per released task it has not started: `ready to start`, or `waits on` each of the open dependencies by slug, another machine's ownership, a pause, a refused start with its text and the minutes to its retry, and `max_runs`.

  Every growing time renders in minutes, so the pane's poll still backs off.
- **`lyx beam status`** is a new operator verb that prints, for the prime's dobby and each local beam's, the structured `dobbyengine.View` and the same rendered lines.
  An entry whose view does not read carries its error, and the verb fails naming it.
- **Dobby's log** gets one line per task start and per pass spawned, `<time> start <slug>` or `<time> pass <slug> continue|full|full-no-reopen`, through a new `Deps.Log` seam that the hub wiring points at the process's stdout, which is `dobby.log`.
- **The start record** keeps the last refusal text (`StartState.Refusal`, an optional key), so the view can say why a start was refused after its notice went out.

## Design decisions

- `dobbyengine.BuildView` is a pure function over the liveness, state record, beam.yaml, run listing, board listing and pause coverage.
  It derives the wait reasons from the same predicates the start step uses: `readyToStart` is now `released && !heldByEntry`, and `max_runs` comes from the shared `slotsTaken` count, walked in board order as `startReady` walks it.
  So the view cannot give a reason the scheduler does not act on.
- `TaskView.DependenciesDone` becomes `PendingDependencies []string`, and the parent and owner are now read for every released task, not only for those whose dependencies are done.
  A task that waits on dependencies has to be placed under its dobby before it can be shown.
  The parents are cached per slug, as before.
- The hub reads of runs, landings and tasks move from `internal/beamcli` to exported functions in `internal/battencli` (`dobbyreads.go`: `DobbyRuns`, `DobbyLandings`, `DobbyTasks`, the `DobbyParents` cache and `DobbyViewReader`).
  `beamcli` imports `battencli`, so the overview could not reach the reads where they were.
  `battencli` already exports the other wirings dobby reuses.
  Their bodies are unchanged apart from the dependency field.
- The rendering lives beside the overview's own rendering in `battencli` (`overviewdobby.go`), and `lyx beam status` reuses it, so the pane and the verb print the same lines.
- `lyx beam status` covers every local dobby, the same set `lyx beam start` considers, rather than only the acting worktree's.
- Known bound: a first pass in flight that has not yet written a status file is in no file, so for that moment its task reads as ready and its slot as free.

## Docs

The package docs of `dobbyengine`, `beamcli` and `battencli` describe the view, the verb, the log line and the moved reads.
`docs/overview.md` lists `lyx beam status` and the overview's dobby section, and the orch role stencil names `lyx beam status`.

## Testing

- `TestBuildView` (tier0) covers a ready task, each entry reason, a refused start before and after its backoff and after an entry change, `max_runs` in board order and at 0, done and owned-elsewhere runs taking no slot, other worktrees' tasks, drafts and projects left out, and a dead dobby.
- `TestDobbyLines` and `TestOverviewSource_DrawsTheDobbyBelowTheRuns` (tier0) pin the rendered lines and their place in the frame.
- `TestLog_OneLinePerStartAndPass` (tier0) pins the log lines for a full pass, a continue pass and a task start.
  The refusal test now asserts the recorded refusal text.
- `TestStatus_PrintsThePrimesDobbyAndEachLocalBeams` (tier1) runs the verb over a hubforge hub with one beam.
- Tier0 and tier1 over the change's computed scope pass.
