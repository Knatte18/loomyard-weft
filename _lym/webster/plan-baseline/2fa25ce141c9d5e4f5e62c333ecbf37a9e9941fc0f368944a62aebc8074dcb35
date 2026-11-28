# Card 6 — kwinplace-scripts-readers

**Create:**
- `internal/kwinplace/doc.go`
- `internal/kwinplace/testmain_test.go`
- `internal/kwinplace#Rect`
- `internal/kwinplace#Desktop`
- `internal/kwinplace#Output`
- `internal/kwinplace#Tile`
- `internal/kwinplace#Window`
- `internal/kwinplace#Snapshot`
- `internal/kwinplace#Snapshot.ZoneOf`
- `internal/kwinplace#Window.Matches`
- `internal/kwinplace#ApplyOrder`
- `internal/kwinplace#ApplyReport`
- `internal/kwinplace#QueryScript`
- `internal/kwinplace#ApplyScript`
- `internal/kwinplace#ReportLines`
- `internal/kwinplace#ParseQueryReport`
- `internal/kwinplace#ParseApplyReport`
- `internal/kwinplace#TilingOutput`
- `internal/kwinplace#ErrNoTilingOutput`
- `internal/kwinplace#TestRecordings_EveryRecordingReads`
- `internal/kwinplace#TestWindow_Matches`
- `internal/kwinplace#TestApplyScript_AddressesWindowsByInternalID`

**Uses:**
- `internal/kwinplace/query.js`
- `internal/kwinplace/apply.js`
- `internal/kwinplace/testdata/recordings/neve-query/journal.txt`
- `internal/kwinplace/testdata/recordings/neve-query-no-output/journal.txt`
- `internal/kwinplace/testdata/recordings/neve-apply/journal.txt`
- `internal/kwinplace/testdata/recordings/neve-apply-one-row/journal.txt`
- `internal/kwinplace/testdata/recordings/neve-kscreen/kscreen.json`
- `internal/kwinplace/testdata/recordings/neve-kscreen-laptop-off/kscreen.json`

**Intent:** Create `internal/kwinplace` (D3), the only code that reaches KWin and kscreen,
with the fillers of the scripts it loads into KWin and the readers of what they and `kscreen-doctor` report.
This card runs nothing; card 8 runs the scripts.

`doc.go` documents the package: it owns the KWin and kscreen boundary and the placement flow,
reaches KWin only by loading a generated script through `qdbus6`, and reads each script's report from the `plasma-kwin_wayland.service` user journal.

The snapshot types hold what D3 step 1 lists:
`Snapshot` holds `CurrentDesktop`, the `Desktops` (id, name, order), the `Outputs` (name, logical geometry as a `Rect`),
`PerDesktopTiling` (the probe's result), the `Windows` and, per desktop id, the leaf `Tile`s of `rootTile(tilingOutput, desktop)`,
each with `AbsoluteGeometry`, `Padding` and `Windows` (internal ids).
`Window` holds `InternalID`, `PID`, `ResourceClass`, `Caption`, `Desktops` (ids), `Output`, `Minimized`,
`Stacking` (its index in `workspace.stackingOrder`) and `Frame`.
`ApplyOrder` is what the apply script is told: the tiling output's name, the target desktop id, `Rows`, `Cols`,
the zone-ordered internal ids (`""` for an empty zone) and the placed window's internal id,
with the JSON names card 1's `apply.js` reads from its input.
`ApplyReport` holds the target desktop's usable area (`workspace.clientArea` with the placement area), the target desktop's leaf `Tile`s and, per reported window,
its internal id, desktops, zone on the target desktop, whether a tile holds it and its frame.

`Snapshot.ZoneOf` returns the index of the leaf whose `Windows` lists the window on that desktop,
never the window's `tile`, which the report does not carry (D3, Constraints).
`Window.Matches` is D2's match: `PID` equals `pid`, `ResourceClass` equals the named constant `konsoleResourceClass`, `"org.kde.konsole"`,
and `Caption` contains `title`.

`QueryScript` and `ApplyScript` embed card 1's `query.js` and `apply.js` through `//go:embed`
and replace each script's placeholder line's `__LYM_KWINPLACE_INPUT__` with the JSON object it takes:
the run id and the tiling output's name for the query, and the run id with the `ApplyOrder` for the apply (D3).
The placeholder is a named constant.

`ReportLines` keeps the journal lines of one run id, in order, and reports whether the end marker is among them; lines of another run id are ignored (D3).
`ParseQueryReport` and `ParseApplyReport` decode those lines, in the line format card 1's scripts log, into a `Snapshot` and an `ApplyReport`.
A line that does not decode is an error naming the kind; a `no-output` line is `ErrNoTilingOutput` wrapped with the output's name.
`TilingOutput` reads `kscreen-doctor -j` and returns the name of the enabled output with the lowest `priority` (D4); no enabled output is an error.

`testmain_test.go` is `untagged`: it holds only `TestMain`, calling `tmuxkit.Main`.
Tests, all `tier0`:
- `TestRecordings_EveryRecordingReads` walks every directory under `testdata/recordings` that holds a `journal.txt` or a `kscreen.json`
  and runs the reader its file calls for over it, failing on any unexpected error; the `neve-qdbus` answers are card 8's runner's to read.
  - The query snapshot holds the current desktop, every desktop's leaf tiles with their geometry, padding and windows, and each window's internal id;
    `ZoneOf` answers the leaf that lists a window.
  - The lines of the other run id are left out, and the journal cut before its end marker reads as not ended.
  - `neve-query-no-output` reads as `ErrNoTilingOutput` naming the told output.
  - `neve-apply`: the report holds the target desktop's leaf tiles; each bottom-row window under the panel is held by no tile, and its frame ends at the usable area's bottom (D6).
  - `neve-apply-one-row`: every window is held by no tile and sits in its zone's box, cut at the usable area's bottom (D6).
  - `neve-kscreen` names Neve's priority-1 output; `neve-kscreen-laptop-off` names the enabled output, passing over the disabled one;
    the `neve-kscreen` recording cut to its disabled outputs alone (none, on Neve) answers the no-enabled-output error.
- `TestWindow_Matches` rows over hand-built `Window` values, kwinplace's own type:
  the right pid, caption and `org.kde.konsole` match; the right pid with the class `konsole`, with another class, or with another caption does not (D2).
- `TestApplyScript_AddressesWindowsByInternalID` asserts the input Go fills into the apply script:
  its JSON object holds each internal id of the order in zone order, the target desktop id, the placed window's internal id and the named output (D3).
Together they add under 0.3 seconds to one `tier0` run.

The contract tests that drive a real KWin are card 8's, beside the runner that reaches it.
