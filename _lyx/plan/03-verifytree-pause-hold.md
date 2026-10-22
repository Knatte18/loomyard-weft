# Card 3 — verifytree.Verify holds its command under a pause

**Create:**
- `plan:internal/verifytree#PauseHold` -> `type PauseHold struct`
- `plan:internal/verifytree#NewPauseHold` -> `func NewPauseHold(geom pauserecord.Geometry, slugEntryExempt func() bool) *PauseHold`
- `plan:internal/verifytree#MarkerStatePaused` -> `const MarkerStatePaused = "paused"`
- `plan:internal/verifytree#TestVerify_PauseHold` -> `func TestVerify_PauseHold(t *testing.T)`

**Edit:**
- `internal/verifytree#Site`
- `internal/verifytree#Verify`
- `internal/verifytree#Marker`
- `internal/verifytree#MarkerStateWaiting`
- `internal/verifytree/doc.go`

**Uses:**
- `internal/verifytree#newScratch`
- `internal/verifytree#mustVerify`
- `internal/verifytree#ReadMarker`
- `internal/pauserecord#ReadCoverage`
- `internal/pauserecord#Update`

**Intent:**
D4 and the hold half of D5.

`PauseHold` is the told hold, two functions and two timing seams: `PastGrace func() bool`, which reports whether a live entry covering the scope has passed its `Entry.GraceEnd`; `Wait func(ctx context.Context, held func()) error`, which returns once no live entry covers the scope, calls `held` once when it starts waiting, and returns `ctx`'s error when `ctx` ends; `Poll time.Duration`, the interval `Verify` polls `PastGrace` at while the command runs; and `Sleep func(ctx context.Context, d time.Duration) error`, the wait between `Wait`'s record reads (`PATTERN-test-speed`'s injectable time).
`Site` gains the field `Hold *PauseHold`; a nil hold runs exactly as today, which is what every existing test and the `cardverify` and battencli `verifyrun` paths use.

`NewPauseHold` builds the hold every D5 site tells, over `pauserecord`, for `geom`'s record and slug, and never returns nil.
It sets `Poll` to one second and `Sleep` to a timer wait that ends early with `ctx`, today's `WaitUncovered` default, so production keeps today's intervals; a test sets a short `Poll` and a fast `Sleep` on the hold it returns.
A geometry with no record path reads uncovered: `PastGrace` is false and `Wait` returns at once.
`PastGrace` calls `pauserecord.ReadCoverage(geom.RecordPath, slug, time.Now())` and answers covered and now not before the covering entry's grace end; an unreadable record answers false and logs one Warn per change of error, held in the hold under a mutex, so a corrupt file never stops a verify.
`Wait` calls `pauserecord.WaitUncovered(ctx, geom.RecordPath, geom.Slug, ...)` with a `Waiter` whose `Notify` logs the line at Info, whose `Sleep` calls `held` before its first sleep and then the hold's `Sleep`, and whose `SlugEntryExempt` is `slugEntryExempt`.
`WaitUncovered` sleeps only once it has found a covering entry, so `held` runs only for a wait that holds: an unreadable record, whose line `Notify` also logs, returns before any sleep and never calls `held`.
While `slugEntryExempt` reports true, `PastGrace` reads the empty slug, so only a hub entry counts, the exemption `WaitUncovered` already gives a caller whose run holds a parent lock; a nil `slugEntryExempt` never exempts.
`verifytree` importing the leaf `internal/pauserecord` keeps it resolver-free (`PATTERN-told-geometry`): the geometry is told.

`Verify` with a non-nil `site.Hold`, after the dirty and skip checks and before writing its running marker, calls `Hold.Wait`, whose `held` writes the marker with state `MarkerStatePaused` and `WaitStarted` set to now; a `Wait` error returns `verifytree: verify cancelled: <ctx error>` with no record and the marker removed, the caller's cancel during the hold.
Every return after a hold, whether `StatusDirty`, `StatusSkipped`, an error, a cancel or the start over's own result, removes the marker: with a non-nil `site.Hold` only, Verify creates the verify directory and registers `defer os.Remove(p.Marker)` before its first `Wait`, so the restarted dirty and skip checks return with no live `paused` marker left behind, while a nil-hold Verify still registers the removal only after its running marker's write, as today, and never removes a marker on its dirty or skipped return.
Verify's doc comment gains that a held Verify removes its marker on every return, its dirty and skipped returns included.
A `Wait` that held (it called `held`) is followed by a start over from the dirty-tree check, exactly as the stop path below, so a tree dirtied during the pause returns `StatusDirty` and a record written during it naming HEAD's tree skips; the next `Wait` returns at once unless another pause has landed.
Once a `Wait` returns without holding, Verify writes the running marker as today and runs the command under a child context of `ctx`, polling `Hold.PastGrace` every `Hold.Poll` only while the command runs (`PATTERN-daemon-wakeups`); once it answers true, Verify cancels the child context, which `verifyrun.Run` (card 2) turns into SIGTERM on the group and SIGKILL after its delay.
A run stopped this way, while `ctx` itself has not ended, is neither passed nor failed: Verify logs it at Info, keeps that run's log for pruning like any other, sets the marker's state to `MarkerStatePaused`, waits through `Hold.Wait` and starts over from its dirty-tree check, so a tree changed during the pause is verified afresh and a tree whose record now names it skips.
A `ctx` cancel at any point returns the cancel error as today.
A covering entry resumed before its grace end lets the running command finish and record its result.
Bound: Verify stops only the process group of the command it started, and only after the covering entry's grace end.
`MarkerStatePaused` joins the marker's state constants, and `ReadMarker` returns it as is.
The const block's comment ("The two states of a Marker"), held on `MarkerStateWaiting`'s group, and `Marker`'s doc ("State says whether the run still waits for a gate slot") become the three states, the third a Verify held by a pause.

`TestVerify_PauseHold` is table-driven over `newScratch` and a temp pause record, each row building its hold with `NewPauseHold` and setting a short `Poll` and a fast `Sleep` on it, so no row waits out a production interval:
a Verify entered under a live hub entry starts no command while covered, its marker reads `paused` meanwhile, and after the entry is resumed it runs the command once and passes, and `ReadMarker` reports no live marker once Verify has returned;
a Verify entered under a live hub entry, with an untracked file written into the worktree while it is held, returns `StatusDirty` after the resume, runs no command, and `ReadMarker` reports no live marker once Verify has returned;
a command running when a hub entry past its grace end is added gets SIGTERM (its `trap` writes a marker file at an absolute path outside the worktree, so the rerun's dirty check finds a clean tree), the marker reads `paused`, and after the resume the command runs a second time from the dirty check and Verify passes, with the stopped run's `verify-<n>.log` still in the verify directory beside the rerun's, and `ReadMarker` reports no live marker once Verify has returned;
the same stop with an untracked file written into the worktree during the pause returns `StatusDirty` after the resume, runs no second command, and `ReadMarker` reports no live marker once Verify has returned;
the same stop with a record naming HEAD's tree and the command written during the pause returns `StatusSkipped` after the resume, and `ReadMarker` reports no live marker once Verify has returned;
an entry whose grace end is an hour away, added and then resumed while the command runs, lets the one run finish and write the record;
a caller cancel while the Verify is held returns an error wrapping `context.Canceled`, runs no command and leaves no marker;
a slug entry under a hold whose `slugEntryExempt` reports true runs the command at once, and a slug entry past its grace end added while that command runs does not stop it;
a hold built from the zero `pauserecord.Geometry` runs the command at once and never stops it;
an undecodable record runs the command at once, its marker never reads `paused`, and `PastGrace` over it logs one Warn across its polls.
A nil hold is the existing `TestVerify_Scenario` rows, unchanged.

`doc.go` gains a paragraph on the hold: the two functions and two timing seams, the `paused` marker state, the start over from the dirty check after a hold at entry and after the stop at the grace end, the marker's removal on every return after a hold, `NewPauseHold`'s uncovered reading of an unreadable record and its parent-lock exemption.

**ImpactSummary:** `Site` gains an optional field and `Verify` a hold path taken only with it set, so every current caller (four sites, all passing no hold until cards 5 to 7) runs unchanged; one new type, constructor and marker state.
