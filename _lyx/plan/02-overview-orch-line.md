# Card 2 — Orch line in the batten overview

**Create:**
- `plan:internal/battencli#overviewOrchBroken` -> `const overviewOrchBroken = "orch  state unreadable"`
- `plan:internal/battencli#overviewWatcherDown` -> `const overviewWatcherDown = "watcher down"`
- `plan:internal/battencli#orchStatePoll` -> `type orchStatePoll struct`
- `plan:internal/battencli#orchStateCache` -> `type orchStateCache struct`
- `plan:internal/battencli#orchStateCache.Poll` -> `func (c *orchStateCache) Poll(paths orchengine.Paths) orchStatePoll`
- `plan:internal/battencli#orchStateCache.fail` -> `func (c *orchStateCache) fail(modTime time.Time, known bool, err error) orchStatePoll`
- `plan:internal/battencli#overviewThresholds` -> `type overviewThresholds struct`
- `plan:internal/battencli#loadOverviewThresholds` -> `func loadOverviewThresholds(location *lyxcwd.Location) overviewThresholds`
- `plan:internal/battencli#overviewOrchLine` -> `func overviewOrchLine(orch orchStatePoll, thresholds overviewThresholds) (string, bool)`
- `plan:internal/battencli#overviewTokens` -> `func overviewTokens(n int) string`
- `plan:internal/battencli#TestOverviewSource_DrawsTheOrchLine` -> `func TestOverviewSource_DrawsTheOrchLine(t *testing.T)`
- `plan:internal/battencli#TestLoadOverviewThresholds` -> `func TestLoadOverviewThresholds(t *testing.T)`

**Edit:**
- `internal/battencli#renderOverview` -> `func renderOverview(orch orchStatePoll, thresholds overviewThresholds, runs []overviewRun, now time.Time) []string`
- `internal/battencli#overviewSource`
- `internal/battencli#newOverviewSource` -> `func newOverviewSource(prime *lyxcwd.Location, pauseRecord string, thresholds overviewThresholds, now func() time.Time) *overviewSource`
- `internal/battencli#overviewSource.frame`
- `internal/battencli#battenCLI.overviewCmd`
- `internal/battencli#TestRenderOverview`
- `internal/battencli#TestOverviewSource_ReadsThePrimesRunsFromDisk`
- `internal/battencli/cli.go`
- `internal/orchengine/doc.go`
- `cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `plan:internal/orchengine#PeekState`
- `internal/orchengine#LoadConfig`
- `internal/orchengine#Config.Threshold`
- `internal/orchengine#Config.SoftThreshold`
- `internal/orchcli#PrimePaths`
- `internal/fabricengine#ConfigDirs`

**Intent:**
Draw one orch line first in the prime's overview frame (D1, D4), with every new member in a new file `internal/battencli/overvieworch.go` whose header comment says it reads and renders the orch line; `overview.go` keeps the run lines.

The orch state read (D2).
`orchStatePoll` holds `State orchengine.State`, `Found bool` and `Unreadable bool`, the orch counterpart of `statuspane.StatusPoll`.
`orchStateCache.Poll` follows `statuspane.StatusCache.Poll`'s contract over `paths.StatePath`: it stats the file; an absent file resets the cache and returns the zero poll; a modification time equal to the last good decode's returns the cached state without reading; otherwise it calls `orchengine.PeekState(paths)`.
A failed stat, a `PeekState` error (a read or decode failure) is a failure through `orchStateCache.fail`: the first failure in a row returns the last good state (`Found` false when there was none), and a second failure in a row, or a later poll at the same failed modification time, returns `Unreadable`; the next good decode clears the count.
`PeekState` reporting not found after a successful stat (the file removed in between) reads as absent; it is a race guard no deterministic input reaches, so it takes no test row.
`fail` logs the error at Debug (`PATTERN-log-levels`), as the overview's other per-frame failures do.
The cache takes no lock and creates nothing, since `PeekState` does neither.

The thresholds (D3).
`overviewThresholds` holds `Soft`, `Hard int` and `Known bool`.
`loadOverviewThresholds` calls `orchengine.LoadConfig(fabricengine.ConfigDirs(location), "orch")` and returns `SoftThreshold()` and `Threshold()` with `Known` true; on a load error it logs at Warn and returns the zero value, which draws `-` for both thresholds, so a broken `orch.yaml` never refuses the verb.
The verb's non-refusal is structural: `loadOverviewThresholds` returns no error for `battenCLI.overviewCmd` to refuse on.
Bound: the verb loads it once at start, so a threshold edit shows only once the runs strand is replaced (D3).

The line (D4).
`overviewTokens` renders a count under 1000 as the plain number and a larger one as the count divided by 1000, rounded down, with a `k` suffix (`999`, `1k`, `212k`).
`overviewOrchLine` returns false when the poll is neither `Found` nor `Unreadable` (no state file: no orch line), `overviewOrchBroken` when `Unreadable`, and otherwise `orch  <tokens> / soft <soft> / hard <hard>  <phase>  cycles <n>`: tokens `-` when `LastContextKnown` is false, each threshold through `overviewTokens` or `-` when not `Known`, the persisted `Phase` as is, `CycleCount`, and `  watcher down` (`overviewWatcherDown`) appended when `WatcherExit` is non-empty.
The line is not padded into the run columns.

`renderOverview` takes the orch poll and the thresholds and prepends the orch line, when there is one, to the run lines it renders unchanged, `no runs in flight` included.
`overviewSource` gains the prime's orch paths (`orchcli.PrimePaths(prime)`, set by `newOverviewSource`), an `orchStateCache` and the thresholds `newOverviewSource` is now told.
`overviewSource.frame` polls the orch cache first on every frame and passes it to `renderOverview`; on its `runs unreadable` early return it prepends the same orch line, so the line draws on every frame.
`battenCLI.overviewCmd` calls `loadOverviewThresholds(location)` after the prime refusal and passes the result to `newOverviewSource`.
Its `Short` becomes `draw the orch's context and the runs in flight in a pane (run by the prime's runs strand)`, and its `Long` names the orch line first: the orch session's context reading against the soft and hard thresholds, its cycle phase and cycle count, then one line per batten run in flight; no test asserts the old wording (grepped).
`internal/orchengine/doc.go`'s `start` bullet says the overview window shows the orch's context line above the runs in flight.
The comments that describe the overview as runs only name the orch line too: `overview.go`'s file header, the doc comments of `renderOverview` (its "no run to show renders the one line" sentence holds only with no orch line), `overviewSource`, `overviewSource.frame` and `battenCLI.overviewCmd`, and `internal/battencli/cli.go`'s package-doc sentence on `lyx batten overview`.

Tests.
`TestRenderOverview` gains `orch orchStatePoll` and `thresholds overviewThresholds` fields, zero in every existing row, so those rows keep their frames with no orch line (acceptance: no state, parent's frame).
New rows, with thresholds `{Soft: 300000, Hard: 400000, Known: true}` unless stated:
- a found idle state with 212999 known tokens and cycle count 3 above two runs: first line `orch  212k / soft 300k / hard 400k  idle  cycles 3`, then the two run lines as before;
- a state with `LastContextKnown` false and phase `compacting`: `orch  - / soft 300k / hard 400k  compacting  cycles 0`;
- 999 known tokens: `orch  999 / soft 300k / hard 400k  idle  cycles 0`;
- 1000 known tokens: `orch  1k / soft 300k / hard 400k  idle  cycles 0`;
- a state with `WatcherExit` set: the line ends `  watcher down`;
- unknown thresholds: `orch  212k / soft - / hard -  idle  cycles 3`;
- an `Unreadable` poll above a run: `orch  state unreadable`, then the run line.
`TestOverviewSource_ReadsThePrimesRunsFromDisk` passes the zero `overviewThresholds{}` to `newOverviewSource`; it writes no orch state, so its frames are unchanged.
`TestOverviewSource_DrawsTheOrchLine` builds a prime with `locationkit.Location`, no runs, thresholds `{300000, 400000, true}`, and writes the orch state file at `orchcli.PrimePaths(prime).StatePath` through `writeOverviewFile` with explicit modification times; frame by frame:
1. no state file: `no runs in flight` alone, and afterwards the orch paths' `Dir` does not exist;
2. `{not json` as the first state file: `no runs in flight` alone (a first failure with no good state draws no line);
3. a good state (212500 known tokens, idle, 3 cycles) at a new modification time: the orch line then `no runs in flight`;
4. the same file rewritten with 100000 known tokens at frame 3's modification time: frame 3's orch line unchanged (an unchanged modification time is not re-read);
5. `{not json` at a new modification time: the last good orch line still draws (first failure);
6. the same file again: `orch  state unreadable`, then `no runs in flight` (a repeat of the failed modification time);
7. a good state with `WatcherExit` set, at a new modification time: the line ends `  watcher down`;
8. `{not json` at a new modification time: the `watcher down` line still draws (the good decode cleared the count);
9. after every frame so far, `StateLockPath` does not exist (no lock taken or created);
10. the orch `Dir` removed and a regular file written at its path, so the stat fails with no modification time: `orch  state unreadable` (a second failure in a row, at no repeated modification time);
11. that file removed and a good state written again, then a regular file written where the prime's runs directory belongs, `filepath.Dir(filepath.Dir(shedrun.SeedFile(prime, "x")))`: the orch line, then `runs unreadable`;
12. the state file removed: `runs unreadable` alone.
`TestLoadOverviewThresholds` is table-driven over a `locationkit.Location` prime: no `orch.yaml` returns the template's `{300000, 400000, true}` and logs no Warn; an `orch.yaml` at `configengine.ConfigFile(prime.AnchorPath(), "orch")` holding `cycle_mode: bogus` returns the zero value, unknown, and logs one Warn line, captured with `logcapture.Capture`.
It does not call `t.Parallel`, since `logcapture` redirects the process-global logger.
Raise `internal/battencli`'s row in `cmd/lyx/testdata/test-budget.yaml` from 81 to 83 for the two new top-level tests: the source test pins a second file the source polls under its own two-strike contract, and the loader is its own function with its own failure branch.

**ImpactSummary:** confined to the overview in `internal/battencli`: `renderOverview` and `newOverviewSource` are re-signed with their two production and two test callers updated on this card; one help text, the overview's doc comments, one sentence each in the batten and orch package docs, and one test-budget row change.

**Commit:** 2: batten overview: orch context line above the runs
