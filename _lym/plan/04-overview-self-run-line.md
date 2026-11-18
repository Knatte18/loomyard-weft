# Card 4 — overview-self-run-line

**Create:**
- `plan:internal/battencli#selfRunReader` -> `type selfRunReader struct`
- `plan:internal/battencli#newSelfRunReader` -> `func newSelfRunReader(worktree *lymcwd.Location) *selfRunReader`
- `plan:internal/battencli#selfRunReader.line` -> `func (r *selfRunReader) line() (string, bool)`
- `plan:internal/battencli#checklistWindowName` -> `func checklistWindowName(st *reedengine.ReedState) string`
- `plan:internal/battencli#TestOverviewSource_DrawsTheSelfRunLine` -> `func TestOverviewSource_DrawsTheSelfRunLine(t *testing.T)`

**Edit:**
- `internal/battencli#overviewSource`
- `internal/battencli#newOverviewSource`
- `internal/battencli#overviewSource.frame`
- `internal/battencli#battenCLI.overviewCmd`
- `docs/overview.md`

**Uses:**
- `internal/battencli#overviewStage`
- `internal/battencli#writeOverviewFile`
- `internal/battencli#writeOverviewStatus`
- `internal/battencli#minutesAgo`
- `internal/battencli#overviewNow`
- `internal/battencli#orchThresholds`
- `internal/battencli#TestOverviewSource_DrawsTheOrchLine`
- `internal/loomengine#LoomStatusStrandName`

**Intent:**
D3: `lym batten overview` draws one line for its worktree's own loom run, naming the window the checklist is in.

`selfRunReader` holds the worktree location and a `*statuspane.StatusCache`; it is not safe for concurrent use.
`line` reads, on each frame, without any lock but reed's:
1. the seed through `shedrun.ReadSeed(worktree, shedrun.SelfRunID)`; an error or an absent seed returns false;
2. the status through its cache's `Poll(shedrun.StatusFile(worktree, shedrun.SelfRunID))`; a poll that is not `Found`, `Unreadable` included, or whose state is `shedengine.StateDone`, returns false;
3. reed's state through `reedengine.LoadState(filepath.Join(worktree.AnchorPath(), lymdirs.DotLymDirName))`, whose `state.ReadJSON` takes reed's shared read lock on `reed.json.lock`.
The line is `loom <recipe>  <stage>  <state>`, the stage being `overviewStage(seed.Recipe, status.CurrentProducer)` and the state `string(status.State)`, so running, paused, blocked and failed all show.
When reed's state reads and `checklistWindowName` returns a name, the line ends with `  checklist in window <name>`; a failed read, absent state or empty name leaves the suffix off.
No unreadable file is an error of the overview: each failure leaves the line or its suffix out and logs at Debug.

`checklistWindowName` finds the first strand of `st` that `agentname.Matches` addresses as `loomengine.LoomStatusStrandName` and returns "" when there is none or `st` is nil.
It returns the strand's `Display.Window` when that is non-empty and `st.WindowGroups` records the group as `Shared`, else `loomengine.LoomStatusStrandName`, the rule reed names the window by at launch (`createOwnWindowPaneLocked` in `internal/reedengine/spawn.go`, `planOwnWindowPlacement` in `internal/reedengine/ownwindow.go`).
So the name stays right at `window_share_max` 0, for a strand added while the seed was unreadable, and for one that predates card 1.

`overviewSource` gains a `selfRun func() (string, bool)` field, nil drawing no line; `newOverviewSource` sets it to `newSelfRunReader(worktree).line`.
`frame` draws the dobby's lines, then the self-run line, then the orch lines, a blank line between each two sections present, as between the dobby and the orchs today.
A worktree with no seed, as the prime's in `TestOverviewSource_DrawsTheOrchLine`, draws no line, so that test's frames stay as they are.
`overviewCmd`'s `Long` and `Short` texts name the line, `overview.go`'s file header names the section between the dobby's lines and the orch lines, and `docs/overview.md`'s batten entry for the hidden `lym batten overview` says it draws the worktree's own loom run, unless done, with its recipe, row, state and the checklist's window.

`TestOverviewSource_DrawsTheSelfRunLine` is a tier 0 table test, added in `overview_test.go`, over a fresh `locationkit` beam worktree `beam-b` per row, built with `newOverviewSource(prime, beam, orchThresholds, ...)` at `overviewNow`, with no orch state file and no dobby unless a row names them, so the other rows' frames hold the self-run line alone or nothing.
Each row writes the seed through `shedrun.WriteSeed` with `shedrun.RecipePawl`, the status through `writeOverviewStatus` with `CurrentProducer` `Beam-Await`, and `reed.json` under the worktree's anchor `.lym` directory through `writeOverviewFile`, holding a strand named `tst:beam-b:loom-status`.
The want line is built from `overviewStage(shedrun.RecipePawl, "Beam-Await")`.
Rows, asserting the exact frame lines:
- running, with the strand in group `pawl` and `WindowGroups` holding `pawl` as shared: the line ending `checklist in window pawl`;
- paused, blocked and failed with the same reed state: the same line with that state;
- running, group `pawl` recorded as not shared: the line ending `checklist in window loom-status`;
- running, the strand with no group: the line ending `checklist in window loom-status`;
- running, no `reed.json`: the line with no suffix;
- running, `reed.json` holding no loom status strand: the line with no suffix;
- running, garbage bytes in `reed.json`: the line with no suffix;
- no seed: no line;
- garbage bytes as the seed: no line;
- garbage bytes as the status, polled twice: no line on either frame;
- done: no line;
- running with an orch state for `beam-b` written through `writeOverviewFile` and `source.dobby` set to a func returning one fixed line: exactly the dobby line, `""`, the self-run line, `""`, then the orch line, which pins the order and both blank lines;
- done with the same orch state and dobby: the dobby line, `""` and the orch line, so the rest renders as before.

**ImpactSummary:** `overviewSource` gains one field set in its one constructor; `frame`'s output changes only for a worktree whose own run is seeded and not done, so the orch-line test's frames and the overview's other callers are unaffected.

**Commit:** 4: batten overview: show the worktree's own loom run and the checklist's window
