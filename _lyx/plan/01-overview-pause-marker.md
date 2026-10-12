# Card 1 — overview-pause-marker

**Edit:**
- `internal/battencli#overviewRun`
- `internal/battencli#overviewLineOf`
- `internal/battencli#overviewSource`
- `internal/battencli#newOverviewSource` -> `func newOverviewSource(prime *lyxcwd.Location, pauseRecord string, now func() time.Time) *overviewSource`
- `internal/battencli#overviewSource.frame`
- `internal/battencli#battenCLI.overviewCmd`
- `internal/battencli#TestRenderOverview`
- `internal/battencli#TestOverviewSource_ReadsThePrimesRunsFromDisk`
- `internal/battencli/cli.go`
- `docs/overview.md`

**Create:**
- `internal/battencli#overviewPausedFormat`

**Uses:**
- `internal/pauserecord#Read`
- `internal/pauserecord#Record.Covering`
- `internal/hubgeom#PauseGeometry`
- `internal/statuspane#FormatMinutes`

**Intent:**
D1: the prime's runs overview marks a run a live pause covers as paused, with the minutes since the pause began.

- `overviewRun` gains a `PausedAt time.Time` field: the `PausedAt` of the live pause entry covering the run, zero when none covers it.
- `overviewLineOf`: when `run.PausedAt` is non-zero and the state cell it computed is `running`, `paused` or `starting` (`overviewStarting`), the state cell becomes `fmt.Sprintf(overviewPausedFormat, overviewSince(run.PausedAt, now))`.
  `statuspane.FormatMinutes` already floors a negative duration at zero, so a `PausedAt` after the poll's clock renders `paused 0m` with no new branch.
  A covered run in `blocked`, `awaiting` or `failed` keeps its state, and an uncovered batten-paused run keeps the bare `paused` it shows today.
  Only the state cell changes, so the column padding and `fitOverview` are untouched.
- `overviewSource` gains a `pauseRecord string` field, the pause record's path; `newOverviewSource` takes it as its new second parameter.
- `overviewSource.frame` reads the record once per poll with `pauserecord.Read(s.pauseRecord)`, before the slug loop.
  A read or decode error logs one `logger.Debug` line (`"batten: reading the pause record for the overview failed"`, with `err`) and continues with the empty record, so a broken record never blanks the pane.
  For each polled run it sets `run.PausedAt` from `record.Covering(slug)`, which returns the oldest live entry covering the slug (a hub entry or the slug's own), and ignores resumed entries.
  The overview still writes nothing; `pauserecord` replaces the file atomically, so the lock-free read is safe.
- `overviewCmd`'s `RunE` resolves `hubgeom.PauseGeometry(location)` after the reed geometry, writing the error envelope on failure as its other resolution steps do, and passes `geometry.RecordPath` to `newOverviewSource`.
  It reaches that call only after `RefuseUnlessPrime` and `hubgeom.ReedGeometry` passed, and `ReedGeometry` fails on exactly the same `.git` read `PauseGeometry` does, so the new failure branch is a fail-loud wiring guard with no row of its own.

Tests:

- `TestRenderOverview` gains rows over built runs at `overviewNow`:
  a running run with `PausedAt` 7 minutes ago shows `paused 7m` in its state cell;
  a batten-paused run (state `paused`) with `PausedAt` set shows `paused <minutes>`;
  a seeded run with no status (`starting`) with `PausedAt` set shows `paused <minutes>` with stage `-`;
  one row holding a `blocked`, an `awaiting` and a `failed` run, each with `PausedAt` set, shows each run's own state;
  an uncovered batten-paused run shows the bare `paused`;
  a running run whose `PausedAt` lies 5 minutes after `overviewNow` shows `paused 0m`.
- `TestOverviewSource_ReadsThePrimesRunsFromDisk` passes a record path under its own `t.TempDir()` to `newOverviewSource`; the path holds no file for the existing three frames, which stay as they are (a missing record is the empty record).
  After the third frame it rewrites run-b's status as a decodable running status with a modification time other than the `minutesAgo(40)` of the undecodable write (for example `minutesAgo(35)`), since `StatusCache.Poll` keeps reporting a failure while the file's modification time equals the failed one, and adds frames:
  4. the record holds a live slug entry for `run-a` paused 10 minutes before `overviewNow` and a live hub entry paused 4 minutes before: run-a shows `paused 10m` (the older entry), and run-b and strict show `paused 4m` (the hub entry covers every run);
  5. the record file holds `{not json`: every run shows its batten state and the frame still draws.
  A resumed-only record is not given a frame: `Record.Covering` reads live entries only, and `pauserecord`'s own `TestCovering` pins that.
  The record is written as JSON of `pauserecord.Record` through `writeOverviewFile`.

Docs: the overview paragraph of the package comment in `internal/battencli/cli.go` and the `lyx batten overview` sentence in `docs/overview.md`'s batten bullet each gain that a run a live pause covers shows `paused <minutes>` in its state cell.

**ImpactSummary:** One package: `newOverviewSource`'s two callers are `overviewCmd` and the source test, both on this card; the render change touches only the state cell, so no other overview row moves.

**Commit:** 1: batten overview: show a pause-covered run as paused with its minutes
