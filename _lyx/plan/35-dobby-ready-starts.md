# Card 35 — dobby-ready-starts

**Edit:**
- `plan:internal/dobbyengine#Deps`
- `plan:internal/dobbyengine#daemon.tick`
- `plan:internal/dobbyengine#StartState`
- `plan:internal/dobbyengine#dobbyengineAllowedImports`
- `internal/dobbyengine/doc.go`

**Create:**
- `plan:internal/dobbyengine#TaskView` -> `type TaskView struct`
- `plan:internal/dobbyengine#readyToStart` -> `func readyToStart(task TaskView, here string, covered bool) bool`
- `plan:internal/dobbyengine#daemon.startReady` -> `func (d *daemon) startReady(ctx context.Context, now time.Time) error`
- `plan:internal/dobbyengine#nextStartTry` -> `func nextStartTry(st StartState, now time.Time, probe time.Duration) StartState`
- `plan:internal/dobbyengine#TestReadyToStart` -> `func TestReadyToStart(t *testing.T)`
- `plan:internal/dobbyengine#TestStartReady_RefusalsAndBackoff` -> `func TestStartReady_RefusalsAndBackoff(t *testing.T)`

**Uses:**
- `plan:internal/boardengine#IsEmptyStatus`
- `plan:internal/dobbyengine#Config`

**Intent:**
Start the beam's ready tasks from the board (D5): a task waits as `draft` until released, and dobby starts it from an empty status.
`Deps` gains the board read (each entry as a `TaskView`: slug, kind, status, whether every `depends_on` is done, its parent worktree through `hubgeom.ParentOf`, its effective owner and a fingerprint of the entry), the `beam.yaml` load of the dobby's worktree layer stack (`max_runs`), and the read of whether the first pass left a status file.
`dobbyengineAllowedImports` gains `internal/boardengine`, imported for `IsEmptyStatus`.
`readyToStart` holds when the entry is a task, its status is empty (`boardengine.IsEmptyStatus`), every dependency is done, its parent is the dobby's worktree, its owner is this machine or none, and no pause entry covers the hub or the slug.
`startReady` reads the local board on a write of `board.json` and on the probe tick, and starts each ready task while fewer than `max_runs` runs it drives are in flight (halted ones included, ones named owned elsewhere not); `max_runs: 0` starts nothing.
Starting is the run's first pass, a full pass through `SpawnPass`, which claims the task as batten does; a slug with a pass in flight is never started twice.
A first pass refused at pre-run for a covering pause (`PassExitCovered`) is no refused start: no notice, no marker, no backoff, and the start waits for the lift.
Any other first pass that exits non-zero and leaves no status file, a refusal as owned elsewhere included (no owner pause, since there is no run to pause), is a refused start: one notice per refusal text to the nearest live orch (`<slug>: dobby could not start the task: <text>`), and the start is retried from one probe interval, doubling up to an hour (`nextStartTry`), and at once when the entry's fingerprint changes; the backoff and the texts noticed are kept in `StartState`, cleared when a pass claims the task or its board entry changes.
Bound: a task is started only from an empty status, by its parent's dobby or a hand-run `lyx batten run`, and never twice while a pass of it is in flight; across machines only the owner machine's dobby of the parent runs; a release written on another machine reaches this dobby when this machine's board is next pulled.
The doc states the start rule and its bounds.
Tests (tier0, fakes, parallel): `TestReadyToStart` rows: an empty-status task with done dependencies under the dobby's worktree starts; a draft task, one with an open dependency, one under a covering pause, one owned elsewhere and one whose parent is another beam do not. `TestStartReady_RefusalsAndBackoff` rows: a fourth ready task past `max_runs: 3` waits; a refused first pass notices once and retries after the probe interval, then after twice it, and at once after an entry change; a covered first pass notices nothing and sets no backoff; an owned-elsewhere first pass with no status file is a refused start with one notice; a run marked owned elsewhere leaves a slot under `max_runs` free; two refusals with different texts send two notices, and a pass that claims the task or a change of its entry clears the noticed texts, so the same text after it notices again.

**ImpactSummary:** The engine's start step and its board seams; filled by the beam verb's wiring.
