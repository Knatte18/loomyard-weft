# Card 5 — kwinplace-scripts-readers

**Create:**
- `internal/kwinplace/doc.go`
- `internal/kwinplace/testmain_test.go`
- `internal/kwinplace/testdata/recordings/neve-query/journal.txt`
- `internal/kwinplace/testdata/recordings/neve-query/source.yaml`
- `internal/kwinplace/testdata/recordings/neve-query-no-output/journal.txt`
- `internal/kwinplace/testdata/recordings/neve-query-no-output/source.yaml`
- `internal/kwinplace/testdata/recordings/neve-apply/journal.txt`
- `internal/kwinplace/testdata/recordings/neve-apply/source.yaml`
- `internal/kwinplace/testdata/recordings/neve-apply-one-row/journal.txt`
- `internal/kwinplace/testdata/recordings/neve-apply-one-row/source.yaml`
- `internal/kwinplace/testdata/recordings/neve-kscreen/kscreen.json`
- `internal/kwinplace/testdata/recordings/neve-kscreen/source.yaml`
- `internal/kwinplace/testdata/recordings/neve-kscreen-laptop-off/kscreen.json`
- `internal/kwinplace/testdata/recordings/neve-kscreen-laptop-off/source.yaml`
- `plan:internal/kwinplace#Rect` -> `type Rect struct`
- `plan:internal/kwinplace#Desktop` -> `type Desktop struct`
- `plan:internal/kwinplace#Output` -> `type Output struct`
- `plan:internal/kwinplace#Tile` -> `type Tile struct`
- `plan:internal/kwinplace#Window` -> `type Window struct`
- `plan:internal/kwinplace#Snapshot` -> `type Snapshot struct`
- `plan:internal/kwinplace#Snapshot.ZoneOf` -> `func (s Snapshot) ZoneOf(windowID, desktopID string) (int, bool)`
- `plan:internal/kwinplace#Window.Matches` -> `func (w Window) Matches(pid int, title string) bool`
- `plan:internal/kwinplace#ApplyOrder` -> `type ApplyOrder struct`
- `plan:internal/kwinplace#ApplyReport` -> `type ApplyReport struct`
- `plan:internal/kwinplace#QueryScript` -> `func QueryScript(runID, tilingOutput string) string`
- `plan:internal/kwinplace#ApplyScript` -> `func ApplyScript(runID string, order ApplyOrder) string`
- `plan:internal/kwinplace#ReportLines` -> `func ReportLines(journal []byte, runID string) ([]string, bool)`
- `plan:internal/kwinplace#ParseQueryReport` -> `func ParseQueryReport(lines []string) (Snapshot, error)`
- `plan:internal/kwinplace#ParseApplyReport` -> `func ParseApplyReport(lines []string) (ApplyReport, error)`
- `plan:internal/kwinplace#TilingOutput` -> `func TilingOutput(kscreenJSON []byte) (string, error)`
- `plan:internal/kwinplace#ErrNoTilingOutput` -> `var ErrNoTilingOutput = errors.New("kwinplace: KWin has no output of the tiling screen's name")`
- `plan:internal/kwinplace#TestRecordings_EveryRecordingReads` -> `func TestRecordings_EveryRecordingReads(t *testing.T)`
- `plan:internal/kwinplace#TestWindow_Matches` -> `func TestWindow_Matches(t *testing.T)`
- `plan:internal/kwinplace#TestApplyScript_AddressesWindowsByInternalID` -> `func TestApplyScript_AddressesWindowsByInternalID(t *testing.T)`

**Intent:** Create `internal/kwinplace` (D3), the only code that reaches KWin and kscreen,
with the scripts it loads into KWin and the readers of what they and `kscreen-doctor` report.
This card runs nothing; card 7 runs the scripts.

`doc.go` documents the package: it owns the KWin and kscreen boundary and the placement flow,
reaches KWin only by loading a generated script through `qdbus6`, and reads each script's report from the `plasma-kwin_wayland.service` user journal.

The snapshot types hold what D3 step 1 lists:
`Snapshot` holds `CurrentDesktop`, the `Desktops` (id, name, order), the `Outputs` (name, logical geometry as a `Rect`),
`PerDesktopTiling` (the probe's result), the `Windows` and, per desktop id, the leaf `Tile`s of `rootTile(tilingOutput, desktop)`,
each with `AbsoluteGeometry`, `Padding` and `Windows` (internal ids).
`Window` holds `InternalID`, `PID`, `ResourceClass`, `Caption`, `Desktops` (ids), `Output`, `Minimized`,
`Stacking` (its index in `workspace.stackingOrder`) and `Frame`.
`ApplyOrder` is what the apply script is told: the tiling output's name, the target desktop id, `Rows`, `Cols`,
the zone-ordered internal ids (`""` for an empty zone) and the placed window's internal id.
`ApplyReport` holds the target desktop's usable area (`workspace.clientArea` with the placement area) and, per reported window,
its internal id, desktops, zone on the target desktop, whether a tile holds it and its frame.

`Snapshot.ZoneOf` returns the index of the leaf whose `Windows` lists the window on that desktop,
never the window's `tile`, which the report does not carry (D3, Constraints).
`Window.Matches` is D2's match: `PID` equals `pid`, `ResourceClass` equals the named constant `konsoleResourceClass`, `"org.kde.konsole"`,
and `Caption` contains `title`.

`QueryScript` writes the query script of D3 step 1 with the tiling output's name filled in.
It reports only `normalWindow` windows.
Its per-desktop tiling probe compares `rootTile(output, d)` across two desktops and reports whether they differ.
When `workspace.screens` holds no output of the told name it logs a `no-output` line naming it, then its end marker.
`ApplyScript` writes the apply script of D3 step 3 and D6:
- it addresses every window by the internal id `ApplyOrder` names;
- it sets the placed window's `desktops` to the target desktop alone and calls `workspace.sendClientToScreen` with the tiling output;
- on `rootTile(output, target)` it releases the tiled windows, removes the root's children, splits columns then rows
  and sets each tile's `relativeGeometry`, as `set-tiling-grid.sh` does;
- it calls `manage` on each zone's window in order;
- for each zone whose bottom edge runs below `workspace.clientArea` with the placement area, it releases the window
  and sets its `frameGeometry` to the zone's box inset by the tile's padding, cut at the usable area's bottom;
- it reports the usable area and every window's desktops, its zone (the leaf whose `windows` lists it), whether a tile holds it and its frame.
Every line either script logs carries the marker `lym-kwinplace`, the run id and a kind, then one JSON object.
The last line is the end marker of that run id.

`ReportLines` keeps the journal lines of one run id, in order, and reports whether the end marker is among them; lines of another run id are ignored (D3).
`ParseQueryReport` and `ParseApplyReport` decode those lines into a `Snapshot` and an `ApplyReport`.
A line that does not decode is an error naming the kind; a `no-output` line is `ErrNoTilingOutput` wrapped with the output's name.
`TilingOutput` reads `kscreen-doctor -j` and returns the name of the enabled output with the lowest `priority` (D4); no enabled output is an error.

The recordings are taken from Neve's live session, never written by hand (`PATTERN-test-stand-ins`):
- `neve-kscreen`: `kscreen-doctor -j`'s output;
- `neve-kscreen-laptop-off`: the same with `eDP-1` disabled, so a disabled output holds a priority value;
  disabling a screen moves windows, so it is taken only when the operator runs or approves it;
- `neve-query`: the journal of one `QueryScript` run, loaded, run and unloaded with `qdbus6` as the operator's `set-tiling-grid.sh` does,
  then read with `journalctl --user -u plasma-kwin_wayland.service --since`;
  it is taken while at least two lym Konsole windows sit on two desktops, and it holds at least one line of another run id;
- `neve-query-no-output`: the journal of one `QueryScript` run told an output name KWin does not have;
- `neve-apply`: the journal of one `ApplyScript` run that places a throwaway Konsole window on a 2-row grid of a desktop that is not current, under Neve's bottom panel;
- `neve-apply-one-row`: the same on a 1-row grid.
  The two apply runs move windows, so they run only when the operator runs or approves them, asked through the parent (Open risks).
The query and `neve-kscreen` recordings are read-only, and the implementer takes them itself where its session reaches the user session bus, otherwise through the parent.
Each `source.yaml` names the command, the date, the machine and the Plasma and KWin versions.

`testmain_test.go` is `untagged`: it holds only `TestMain`, calling `tmuxkit.Main`.
Tests, all `tier0`:
- `TestRecordings_EveryRecordingReads` walks every directory under `testdata/recordings` and runs the reader its files call for over it, failing on any unexpected error.
  - The query snapshot holds the current desktop, every desktop's leaf tiles with their geometry, padding and windows, and each window's internal id;
    `ZoneOf` answers the leaf that lists a window.
  - The lines of the other run id are left out, and the journal cut before its end marker reads as not ended.
  - `neve-query-no-output` reads as `ErrNoTilingOutput` naming the told output.
  - `neve-apply`: each bottom-row window under the panel is held by no tile, and its frame ends at the usable area's bottom (D6).
  - `neve-apply-one-row`: every window is held by no tile and sits in its zone's box, cut at the usable area's bottom (D6).
  - `neve-kscreen` names Neve's priority-1 output; `neve-kscreen-laptop-off` names the enabled output, passing over the disabled one;
    the `neve-kscreen` recording cut to its disabled outputs alone (none, on Neve) answers the no-enabled-output error.
- `TestWindow_Matches` rows over hand-built `Window` values, kwinplace's own type:
  the right pid, caption and `org.kde.konsole` match; the right pid with the class `konsole`, with another class, or with another caption does not (D2).
- `TestApplyScript_AddressesWindowsByInternalID` asserts the values Go fills into the apply script:
  each internal id of the order is addressed, and the placed window's desktops are set to the target id alone and it is sent to the named output (D3).
Together they add under 0.3 seconds to one `tier0` run.

The contract test that drives a real KWin is card 7's, beside the runner that reaches it.
