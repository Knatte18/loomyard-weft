# Card 1 — kwin-probes

**Create:**
- `internal/kwinplace/query.js`
- `internal/kwinplace/apply.js`
- `internal/kwinplace/testdata/recordings/neve-kscreen/kscreen.json`
- `internal/kwinplace/testdata/recordings/neve-kscreen/source.yaml`
- `internal/kwinplace/testdata/recordings/neve-query/journal.txt`
- `internal/kwinplace/testdata/recordings/neve-query/source.yaml`
- `internal/kwinplace/testdata/recordings/neve-query-no-output/journal.txt`
- `internal/kwinplace/testdata/recordings/neve-query-no-output/source.yaml`
- `internal/kwinplace/testdata/recordings/neve-qdbus/load.txt`
- `internal/kwinplace/testdata/recordings/neve-qdbus/load-taken.txt`
- `internal/kwinplace/testdata/recordings/neve-qdbus/loaded-true.txt`
- `internal/kwinplace/testdata/recordings/neve-qdbus/loaded-false.txt`
- `internal/kwinplace/testdata/recordings/neve-qdbus/source.yaml`
- `internal/kwinplace/testdata/recordings/neve-apply/journal.txt`
- `internal/kwinplace/testdata/recordings/neve-apply/source.yaml`
- `internal/kwinplace/testdata/recordings/neve-apply-one-row/journal.txt`
- `internal/kwinplace/testdata/recordings/neve-apply-one-row/source.yaml`
- `internal/kwinplace/testdata/recordings/neve-kscreen-laptop-off/kscreen.json`
- `internal/kwinplace/testdata/recordings/neve-kscreen-laptop-off/source.yaml`

**Intent:** Observe the KWin facts the decision record's Open risks list before any card's code rests on them,
and take every recording the kwinplace tests read (D1 to D4, D6, `PATTERN-test-stand-ins`).
The card adds no Go code and no test:
card 6's `TestRecordings_EveryRecordingReads` reads every journal and kscreen recording it takes,
card 8's `recordedExec` answers with the `neve-qdbus` ones, and card 8's contract test compares them with a live KWin.

The query and apply scripts are written here, since every journal recording is a run of one of them.
Card 6's `QueryScript` and `ApplyScript` embed the two files and fill in their input.
Each script opens with one placeholder line, `const input = __LYM_KWINPLACE_INPUT__;`, which the caller replaces with one JSON object:
- `query.js` takes `runId` and `tilingOutput`;
- `apply.js` takes `runId`, `tilingOutput`, `desktop` (the target desktop id), `rows`, `cols`,
  `zones` (internal ids in zone order, `""` for an empty zone) and `placed` (the placed window's internal id).

`query.js` is the query script of D3 step 1.
It reports only `normalWindow` windows, each with its `internalId`, `pid`, `resourceClass`, `caption`, `desktops`, `output`, `minimized`,
its index in `workspace.stackingOrder` and its `frameGeometry`.
It reports `workspace.currentDesktop`, the desktops (id, name, order), the outputs (name, logical geometry),
and for every desktop the leaf tiles of `rootTile(tilingOutput, desktop)`, each with its `absoluteGeometry`, `padding` and `windows`.
Its per-desktop tiling probe compares `rootTile(output, d)` across two desktops and reports whether they differ.
When `workspace.screens` holds no output of the told name it logs a `no-output` line naming it, then its end marker.

`apply.js` is the apply script of D3 step 3 and D6:
- it addresses every window by the internal id its input names;
- it sets the placed window's `desktops` to the target desktop alone and calls `workspace.sendClientToScreen` with the tiling output;
- on `rootTile(output, target)` it releases the tiled windows, removes the root's children, splits columns then rows
  and sets each tile's `relativeGeometry`, as `set-tiling-grid.sh` does;
- it calls `manage` on each zone's window in order;
- for each zone whose bottom edge runs below `workspace.clientArea` with the placement area, it releases the window
  and sets its `frameGeometry` to the zone's box inset by the tile's padding, cut at the usable area's bottom;
- it reports the usable area, the target desktop's leaf tiles as `query.js` reports them,
  and every window's desktops, its zone (the leaf whose `windows` lists it), whether a tile holds it and its frame.
Every line either script logs carries the marker `lym-kwinplace`, the run id and a kind, then one JSON object.
The last line is the end marker of that run id.

Each script run is loaded, run and unloaded with `qdbus6` as the operator's `set-tiling-grid.sh` does,
and its journal is read with `journalctl --user -u plasma-kwin_wayland.service --since` from just before its load.
Each `source.yaml` names the command, the date, the machine and the Plasma and KWin versions.
The recordings are taken from Neve's live session, never written by hand.

Read-only probes and recordings, taken by the implementer where its session reaches the user session bus, otherwise asked of the operator through the parent:
- `neve-kscreen`: `kscreen-doctor -j`'s output.
- `neve-query`: one `query.js` run told the output `neve-kscreen` ranks priority 1,
  taken while at least two lym Konsole windows sit on two desktops; it holds at least one line of another run id.
  It observes four facts: the output names match `neve-kscreen`'s (D4);
  the per-desktop probe reports two different root tiles (D1);
  every desktop's leaf tiles read with their geometry, padding and windows on a desktop that is not current (D6);
  and each lym Konsole window's resource class reads `org.kde.konsole` (D2).
- `neve-query-no-output`: one `query.js` run told an output name KWin does not have.
- `neve-qdbus`: `load.txt` is a `loadScript` answer, `load-taken.txt` the `-1` of a second load of a plugin name still loaded,
  and `loaded-true.txt` an `isScriptLoaded` answer while the script is loaded.
  `loaded-false.txt` comes from the end-of-run probe: a `query.js` run whose plugin is unloaded right after its `run` call.
  It observes two facts: the script still logs its end marker (D3's end-of-run guarantee),
  and `isScriptLoaded` then reads false once it has left KWin's list (D3's script lock).

Moving probes and recordings, asked of the operator through the parent before any later card starts,
each run only when the operator runs or approves it:
- `neve-apply`: one `apply.js` run that places a throwaway Konsole window, opened on the output `neve-kscreen` ranks second,
  on a 2-row grid of a desktop that is not current, under Neve's bottom panel.
  It observes three facts on a desktop that is not current.
  `rootTile` with `split`, `relativeGeometry` and `manage` builds the grid and fills it (D3, D6).
  Setting `desktops` and `workspace.sendClientToScreen` move a window from another output (D3, D4).
  `frameGeometry`, `clientArea` and a tile's `absoluteGeometry`, `padding` and `windows` read and set as D6's clamp needs.
- `neve-apply-one-row`: the same on a 1-row grid.
- `neve-kscreen-laptop-off`: `kscreen-doctor -j`'s output with `eDP-1` disabled, so a disabled output holds a priority value;
  disabling a screen moves windows.

The card's commit message records each Open-risks fact with the probe that observed it and its outcome.
A probe that contradicts a fact a decision rests on stops the card before its commit:
the implementer sends the probe's command and output to its parent and halts the run, so the parent reopens the decisions resting on that fact.
No later card starts until every recording above exists.
