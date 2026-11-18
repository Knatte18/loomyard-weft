# Card 5 — overview-own-run-line

**Edit:**
- `internal/battencli#overviewSource`
- `internal/battencli#newOverviewSource`
- `internal/battencli#overviewSource.frame`
- `internal/battencli#battenCLI.overviewCmd`

**Extends:**
- `docs/overview.md`

**Create:**
- `plan:internal/battencli#overviewOwnRunFormat` -> `const overviewOwnRunFormat = "run %s  %s  %s  window %s"`
- `plan:internal/battencli#overviewSource.ownRunLine` -> `func (s *overviewSource) ownRunLine() (string, bool)`
- `plan:internal/battencli#TestOverviewSource_DrawsTheOwnRunLine` -> `func TestOverviewSource_DrawsTheOwnRunLine(t *testing.T)`

**Uses:**
- `internal/battencli#overviewStage`
- `internal/battencli#writeOverviewStatus`
- `internal/battencli#writeOverviewFile`
- `internal/battencli#overviewNow`
- `internal/battencli#orchThresholds`

**Intent:**
The beam overview names the worktree's own run, its row, its state and the checklist's window, so the operator sees where pawl stands and finds its window by name (D4).

`overviewSource` gains two fields:
`worktree *lymcwd.Location`, the worktree whose own run the line reads;
and `ownStatus *statuspane.StatusCache`, the cache polling that run's status file.
`newOverviewSource` sets them from its worktree argument and `statuspane.NewStatusCache()`; it is the only builder of an `overviewSource`, so both are always set.

`ownRunLine` reads `shedrun.ReadSeed(s.worktree, shedrun.SelfRunID)` and polls `shedrun.StatusFile(s.worktree, shedrun.SelfRunID)` through `ownStatus`, without locks, like the frame's other lines.
It answers `fmt.Sprintf(overviewOwnRunFormat, recipe, overviewStage(recipe, status.CurrentProducer), status.State, recipe)` and true only while the seed is found and the poll is `Found` with a state other than `shedengine.StateDone`.
The window name is the recipe, the name card 4 gives the checklist's window.
It answers false for a missing seed, a seed that fails to read (logged at Debug), a missing status file, a done run, and a status that fails to read on two polls in a row, which the cache answers `Unreadable`;
on the first failed poll the cache still answers the last good status, so the line stays one more frame.
`frame` opens with that line when there is one, followed by a blank line when the dobby or orch section follows, so it sits above the dobby's lines.
The file header of `internal/battencli/overview.go`, the `overviewCmd` `Short` text and its `Long` help text say the window opens with that line, and `docs/overview.md`'s `lym batten overview` sentence gains it.

`TestOverviewSource_DrawsTheOwnRunLine` (tier0) builds `newOverviewSource(prime, beam, orchThresholds, …)` at `overviewNow`, the beam a `locationkit.Location` of its own, and checks `frame().Lines` in sequence.
Every status write passes a strictly later modification time than the one before, `minutesAgo(N)` with N falling, so the cache reads each one:
- no seed and no status draws nothing;
- a seed with recipe `pawl` written by `shedrun.WriteSeed` and no status file draws nothing;
- a running status written by `writeOverviewStatus` with a current producer of the pawl recipe draws exactly the one line, its stage numbered as `overviewStage` draws it;
- the bytes `{not json` written by `writeOverviewFile` at the status path draw the same line once, then nothing on the next frame;
- a running status again, then an orch state written by `writeOverviewFile` at `orchcli.PrimePaths(beam).StatePath`, draws the line, a blank line, then the beam's orch line;
- a later status with state `done` draws the orch line alone.
A second source over another beam with a running status and the bytes `{not json` as its seed draws nothing.
The existing `TestOverviewSource_DrawsTheOrchLine` and `TestOverviewSource_DrawsTheDobbyAboveTheOrchs` frames hold no seed, so they keep their expected lines.

**ImpactSummary:** Local to the runs strand's pane process in `internal/battencli`: `newOverviewSource` keeps its signature and its two callers, and a worktree with no live seeded run draws exactly today's frame.

**Commit:** 5: The beam overview names the worktree's own run and its checklist window
