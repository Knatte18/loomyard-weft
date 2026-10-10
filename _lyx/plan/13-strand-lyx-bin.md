# Card 13 — strand-lyx-bin

**Edit:**
- `internal/reedengine#Strand`
- `internal/reedengine#StrandStatus`
- `internal/reedengine#composePaneLaunchLine` -> `func composePaneLaunchLine(sh shell.Shell, launchCmd, strandGUID, name, parent string) (line, exe string)`
- `internal/reedengine#Engine.launchStrandLocked`
- `internal/reedengine#Engine.Status`
- `internal/reedengine#TestComposePaneLaunchLine`
- `internal/reedengine/spawn_test.go`
- `internal/reedengine/state_test.go`
- `internal/reedengine/launchscript_integration_test.go`

**Prosa:**
- `internal/reedengine/doc.go`

**Uses:**
- `internal/reedengine#paneBinPrelude`
- `internal/reedengine#executablePath`

**Intent:** Record on each strand the absolute `lyx` path its pane prelude exported (D4), from the same `panebin.go` chokepoint that composes the prelude, so the record and `LYX_BIN` cannot disagree (`PATTERN-pane-binary-resolution`).
`composePaneLaunchLine` also returns the executable path it composed the prelude from, empty when `executablePath` failed and the prelude was dropped.
`launchStrandLocked` stores that value in `Strand.LyxBin`, a persisted `lyxBin,omitempty` field, on every launch and relaunch, since every strand-realizing path funnels through it, so a reed resume relaunching the pane from another `lyx` records that one; a strand recorded before the field existed decodes with none.
`StrandStatus` gains `LyxBin`, which `Status` copies from the record, so a reader of reed's status (orchcli, card 14) sees it without touching `reed.json`.
`TestComposePaneLaunchLine` asserts the returned path beside the line, and the integration-tier launch-script test takes the second return value; `spawn_test.go`'s launch-script test asserts the launched strand's `LyxBin` equals the injected executable and the status test asserts `Status` reports it; `state_test.go`'s round trip adds the field.
The reed package doc's prelude paragraph gains one sentence saying the strand records the path.

**ImpactSummary:** One unexported helper re-signed with its one production caller and one test caller in the same package, two structs gain a field; every other `StrandStatus` literal uses keyed fields and keeps compiling.
