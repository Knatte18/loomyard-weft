# Card 1 — durfmt

**Create:**
- `internal/durfmt#FormatMinutes`
- `internal/durfmt#FormatSeconds`
- `internal/durfmt#TestFormat`
- `internal/durfmt#TestLeafInvariant_StandardLibraryOnly`
- `internal/durfmt/doc.go`
- `internal/durfmt/testmain_test.go`

**Delete:**
- `internal/statuspane#FormatMinutes`
- `internal/statuspane#FormatSeconds`
- `internal/statuspane#TestFormatMinutes`

**Edit:**
- `internal/loomcli#checklistTime`
- `internal/loomcli#checklistGateLines`
- `internal/battencli#overviewSince`
- `internal/statuspane/doc.go`
- `pattern/PATTERN-leaf-packages.md`
- `docs/overview.md`
- `cmd/lyx/testdata/test-budget.yaml`

**Intent:**
Move the status pane's two duration formatters into a new stdlib-only leaf package `internal/durfmt`, so the timing report (card 11) prints every duration in the status pane's format without importing the pane.

`internal/durfmt/durfmt.go` holds `FormatMinutes` and `FormatSeconds` with their bodies and doc comments unchanged from `internal/statuspane/pane.go`, which loses them.
`internal/durfmt/doc.go` is the package doc: `FormatMinutes` is the format of a time still growing, whole minutes as `12m`, `1h05m` or `2d03h`; `FormatSeconds` is the format of a closed span, `45s`, `12m05s` or `1h05m12s` below a day and `2d03h` from a day on; the package imports the standard library only.
`internal/durfmt/durfmt_test.go` holds `TestFormat`, the table of `statuspane`'s `TestFormatMinutes` moved verbatim (every row for both formatters), and `internal/durfmt/leaf_enforcement_test.go` holds `TestLeafInvariant_StandardLibraryOnly`, calling `scankit.AssertImportAllowlist(t, "internal/durfmt")` as `internal/dotgit`'s leaf test does.
`internal/durfmt/testmain_test.go` runs the tests under `tmuxkit.Main`, as every test package must.

The three callers retarget to `durfmt`: `checklistTime` and `checklistGateLines` in `internal/loomcli/checklist.go`, and `overviewSince` in `internal/battencli/overview.go`; their output text is unchanged, so the checklist and overview tests stay green as they are.
`internal/statuspane/doc.go` drops its two formatter paragraphs and says the panes' time formats come from `internal/durfmt`.
Deleting `TestFormatMinutes` also rewords `internal/statuspane/pane_test.go`'s file comment, which names the format boundaries of the two formatters, to name only what the file still pins.
`pattern/PATTERN-leaf-packages.md` gains an `internal/durfmt` section: imports only the standard library.
`docs/overview.md`'s package tree gains an `internal/durfmt/` line (the duration formats the status panes and the timing report share, a stdlib-only leaf) and its `internal/statuspane/` line stops claiming the time formats.
`cmd/lyx/testdata/test-budget.yaml` gains `internal/durfmt: 2` and lowers `internal/statuspane` to 5.

**ImpactSummary:** three call sites in loomcli and battencli change import only; statuspane's exported surface loses two functions with no other caller; output text is byte-identical.

**Commit:** 1: Move the pane duration formatters into internal/durfmt
