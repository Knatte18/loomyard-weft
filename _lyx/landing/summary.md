# Overview window shows its worktree's dobby; add lyx beam status

Nothing showed what a dobby does or whether it lives.
Its log stayed empty unless a tick failed, `lyx beam` had no status verb, and a released task could sit unstarted beside a live dobby with no stated reason (#872).

The overview pane that `lyx batten overview` draws in the prime and in every beam now carries that worktree's dobby below the run lines, with no new window or strand.
It shows whether the dobby is alive (a dead one shows `lyx beam start` as the way forward), the runs it drives with state, stage and minutes since the last pass or a pass in flight, and each released task it has not started with everything that holds it back.
The reasons are open dependencies by slug, another machine's ownership, a covering pause, a refused start's text with its next try, and `max_runs`; a task with none shows as ready.
A task pair's overview draws no dobby section.

`lyx beam status` prints the same lines once, for the prime's dobby and each local beam's, beside the structured view in its envelope.
Dobby also writes one line to its log per task start (`start <slug>`) and per pass spawn (`pass <slug> <kind>`), through a new `Deps.Log` seam; a nil seam logs nothing.

## Design decisions

- `dobbyengine.BuildView` is a pure function over told inputs, and shares `readyToStart` and the `runsInFlight` slot counting with the scheduler, so the view cannot explain a wait the scheduler lacks.
  A done run that still holds its driver lock takes a slot, as in the scheduler.
- `StartState` gains an optional `Refusal` key, written by a refused start and cleared with the rest of the record when a pass claims the task or its entry changes.
  The record is tolerant JSON, so an older binary keeps the key.
- `TaskView` carries open dependencies by slug instead of a done flag, and the board read resolves parent and owner for every task with an empty status, so a task waiting on dependencies is still placed under its dobby.
  The extra `hubgeom.ParentOf` is cached per slug.
- The hub reads of runs and tasks move from `internal/beamcli` to exported functions in `internal/battencli`, because `beamcli` imports `battencli` and the pane could not reach them otherwise.
  Rendering lives once in `battencli` for both the pane and the verb.
- The pane polls the dobby and driver locks by briefly taking them.
  To keep that from making a real acquirer refuse, `sheddriver.DriveVerb` now waits `lock.ProbeGrace` (250 ms) before refusing a held driver lock.
- Times still growing render in whole minutes, so the pane text changes at most once a minute and its poll keeps backing off.
- The dobby section appears only in the prime and in beams listed by `hubgeom.DobbyStart(...).Beams()`.
- Docs are updated in `internal/dobbyengine/doc.go`, `internal/beamcli/doc.go`, `docs/overview.md` and the orch role stencil.

## To check by hand

- `lyx beam status` and the overview pane against a live hub with a running dobby, a stopped one and a refused start; the tests cover the view and rendering, not a live pane.
- Overview wording in a task pair: confirm it draws no dobby section, and that the dobby section in the prime and a beam appears below the run lines.
- `internal/battencli/overview.go:526-543`: the pane's dobby section follows the prime-or-beam rule; check it against a hand-run pane in a pair.
- `internal/battencli/dobbyreads.go:175-187` and `:303-306`: the pane's liveness probes briefly take the dobby lock and each driven run's driver lock; confirm a hand `lyx batten run` or a starting dobby does not refuse because of a poll.
- `internal/battencli/dobbyreads.go:209-258`: `DobbyTasks` lists open dependencies by slug, skips pruned ones and reads parent and owner for every task with an empty status; confirm on a real board.
- `internal/dobbyengine/starts_test.go:109-134`: the refusal text is cleared when a pass claims the task or its entry changes.
- `internal/dobbyengine/view.go:418-430`: a done run holding its driver lock counts toward `max_runs`, so a task never reads ready while the scheduler holds it back.
- `internal/dobbyengine/passes_test.go:200-223`: the log names `full`, `continue` and `full-no-reopen`.
- `internal/battencli/overview.go:491-496`: the overview's doc comment and `Short` name the prime and each beam.
