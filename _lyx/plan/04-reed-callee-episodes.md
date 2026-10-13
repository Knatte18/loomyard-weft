# Card 4 — The reed watch tick's and name-repair loop's callee failures log once per episode

**Create:**
- `internal/reedengine/calleefailures_test.go`
- `plan:internal/reedengine#TestCalleeFailuresLogOncePerEpisode` -> `func TestCalleeFailuresLogOncePerEpisode(t *testing.T)`

**Edit:**
- `internal/reedengine#Engine`
- `internal/reedengine#Engine.liveBoxLocked`
- `internal/reedengine#Engine.readBorderTitleRowLocked`
- `internal/reedengine#Engine.keepZoomLocked`
- `internal/reedengine#Engine.runResizePinsLocked`
- `internal/reedengine#Engine.adoptPaneGenerationLocked`
- `internal/reedengine#Engine.loadOrInitStateLocked`
- `internal/reedengine#Engine.watchNames`
- `internal/reedengine#Engine.repairNames`
- `internal/reedengine#TestLoadOrInitStateLocked`
- `internal/reedengine/doc.go`
- `cmd/lyx/testdata/test-budget.yaml`

**Intent:**
D11 for the `internal/reedengine` sites the audit in `00-overview.md` reaches through a callee of the watch tick or the name-repair loop.

The watch loop calls `reapplyLayout` on every poll or dormant wake, which runs inside `keepZoomLocked` and calls `applyLayoutLockedOpts`, which calls `liveBoxLocked` before its box-equality skip and `readBorderTitleRowLocked` after it.
A fallback box is never live, so the skip is never taken while the size query fails, and each of these Warns repeated on every tick while its tmux call failed;
`keepZoomLocked`'s three Warns repeat on every tick of a zoomed window while their tmux calls fail.
With no skip, `select-layout` and `runResizePinsLocked` also run on every tick, so a pin whose `resize-pane` fails Warns on every tick.
`reapplyLayout` and `repairNames` both call `loadOrInitStateLocked`, which calls `adoptPaneGenerationLocked`, on every tick or wake, and neither saves state:
the generation read's Warn repeats while its `display-message` fails, and the binding clears for a stale generation stamp and for conflicting bindings are made in memory only, so the next tick reloads the same file and logs the same clear again until some persisting op saves it.
`watchNames` logs `reed: name repair pass failed` on every wake while `repairNames` fails, and `repairNames` logs `reed: could not repair pane title` on every wake while a title still drifts and `select-pane` fails.

`Engine` gains a `calleeFailures logger.EpisodeSet` (mutex-guarded, so the watch goroutine and the name-repair goroutine may share it), one episode per key, living as long as the engine:
- `liveBoxLocked`: both fallback Warns call `e.calleeFailures.Fail("live box", …)` with their existing message and fields, so a malformed answer after a failed query stays in one episode, and a parsed answer calls `Recover("live box", "reed: live window size query answers again", "socket", …, "session", …)`.
- `readBorderTitleRowLocked`: a failed readback calls `Fail("border title", …)`, and a readback without an error calls `Recover("border title", "reed: pane-border-status reads back again", …)`.
- `keepZoomLocked`: the unzoom, post-step listing and re-zoom Warns call `Fail` on the keys `unzoom`, `zoom listing` and `re-zoom`, and the matching call succeeding calls `Recover` with `reed: unzooms the strands' window again`, `reed: lists the strands' window after a step again` and `reed: re-zooms the strands' window again`.
- `runResizePinsLocked`: a failed pin calls `Fail("pin:"+pin.PaneID, …)` with its existing message and fields, and a pin without an error calls `Recover("pin:"+pin.PaneID, "reed: pane height pin works again", …, "pane", pin.PaneID)`.
- `adoptPaneGenerationLocked`: the generation read's error calls `Fail("pane generation", …)` with its existing message and fields, and a read without an error calls `Recover("pane generation", "reed: live pane generation reads again", "socket", …, "session", …)`.
- `repairNames`: a failed `select-pane` calls `Fail("title:"+r.GUID, …)` with its existing message and fields, and a successful one calls `Recover("title:"+r.GUID, "reed: pane title repair works again", …, "guid", r.GUID)` before the existing `reed: repaired pane title` Info, which stays.

The two binding clears are no failure, so they get no episode: `Engine` gains a `stateLoadNotices` value, a mutex and the last logged generation pair and cleared set, since both goroutines load state.
`adoptPaneGenerationLocked` logs its `persisted pane bindings were minted against a different tmux session incarnation` line at Warn when the pair of recorded and live generation differs from the last one logged, records it, and logs the line at Debug when it is the same;
a load that finds no mismatch forgets the recorded pair, so a later mismatch Warns again.
`loadOrInitStateLocked` does the same for `cleared strand pane bindings that named a pane another owner already claims`, comparing the sorted cleared strand GUIDs, and a load that clears nothing forgets the recorded set.
The clears themselves are unchanged.

`watchNames` holds a local `var passFailures logger.Episode` for the length of its loop: a pass error calls `passFailures.Fail` with `reed: name repair pass failed` and its fields, and a pass without an error calls `passFailures.Recover("reed: name repair pass works again", …)`.
Every other caller of these helpers (boot, spawn, attach, layout steps) shares the engine's episodes, so a one-shot step's failure inside a watch-tick episode logs at Debug, and its success ends the episode with the Info line.
No layout, zoom or rename decision changes.
`doc.go`'s account of the watch loop's failure handling names the episode rule for these tmux reads and for the name-repair loop.

`TestCalleeFailuresLogOncePerEpisode`, in `calleefailures_test.go`, is serial (it captures the process-global logger with `logcapture.CaptureVerbose` plus `logger.SetVerbosity(2)`) and table-driven, one row per site, each row naming a `fail` and a `heal` step on the fake tmux, a `call` on one engine, the failure message and the recovery message.
Every row calls `call` three times after `fail` and once after `heal`, and asserts exactly one `level=WARN` and two `level=DEBUG` lines carrying the failure message, each DEBUG with `failed_ticks=`, and exactly one `level=INFO` line carrying the recovery message.
Rows, on `newTestEngine` and `installFakeTmux`:
- "live window size query", calling `e.liveBoxLocked(exactSessionWindowTarget(e.SessionName()))`, failing `display-message` with an error and healing it to `220 50`;
- "malformed live window size answer joins the query episode", whose three failing calls alternate a `display-message` error and the answer `abc def`, asserting one WARN across both messages;
- "pane-border-status readback", calling `readBorderTitleRowLocked`, failing `display-message` and healing it to `top`;
- "unzoom", "post-step listing" and "re-zoom", each calling `keepZoomLocked` with a step that returns nil on a window whose `zoomStateFormat` answer is `1 %2` and whose listing holds `%2`: the unzoom row fails every `resize-pane`, the listing row fails `list-panes`, and the re-zoom row fails every second `resize-pane` through `answerFunc` (the re-zoom), each healed by answering the failing verb without an error;
- "pane height pin", calling `e.runResizePinsLocked([]render.Pin{{PaneID: "%2", Height: 10}})`, failing `resize-pane` and healing it;
- "live pane generation read", calling `e.adoptPaneGenerationLocked(&ReedState{})`, failing the `display-message` generation probe and healing it through `answerGenerations` with the session's generation;
- "pane title repair", on `newRepairTestEngine` with one strand whose live title drifts, calling `repairNames(nil)`, failing `select-pane` and healing it, and also asserting one `reed: repaired pane title` INFO line after the heal;
- "name repair pass", driving `watchNames` with an injected `nameRepairTiming` whose `After` fires at once, cancelling after the fourth wake, with `has-session` failing through the first three wakes and healed before the fourth.

`TestLoadOrInitStateLocked` gains `generations` (passed to `answerGenerations` on `installFakeTmux`), `loads` and `wantLogs` (exact line counts by level and text) columns, captures at Debug and runs serially, and two rows that save the state once and call `loadOrInitStateLocked` three times on one engine without saving:
"a stale generation stamp warns once while the clear stays unsaved", whose persisted stamp names an earlier incarnation (`$0|100|1787000000`) of the live session, asserts one `level=WARN` and two `level=DEBUG` `minted against a different tmux session incarnation` lines;
"conflicting bindings warn once while the repair stays unsaved", whose persisted strands `g1` and `g2` both bind `%1`, asserts one `level=WARN` and two `level=DEBUG` `cleared strand pane bindings` lines.

`cmd/lyx/testdata/test-budget.yaml`'s `internal/reedengine` row rises from 261 to 262.

**ImpactSummary:** two fields on `Engine` and eight method bodies in `internal/reedengine`; the log texts are unchanged, so only their level per tick and new recovery lines change, and no layout, zoom, rename or binding-clear decision does.

**Commit:** 4: reed: log a repeating watch-tick or name-repair callee failure once per episode
