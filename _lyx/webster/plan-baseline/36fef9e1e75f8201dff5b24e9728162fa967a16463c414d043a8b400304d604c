# Card 20 — open-window-on-resume

**Edit:**
- `internal/battencli#battenCLI`
- `internal/battencli#battenCLI.battenPreRun`
- `internal/battencli#battenCLI.wire`
- `internal/battencli#battenVerbTexts`
- `internal/battenshed/doc.go`
- `docs/overview.md`

**Create:**
- `internal/battencli#battenCLI.reopenWindow`
- `internal/battencli#TestBattenPreRun_ReopensTheWindowWithNoClientAttached`

**Uses:**
- `internal/reedengine#Engine.ClientAttached`
- `internal/shedadapters#NewCommandProducer`
- `internal/shedadapters#CommandRow`
- `internal/battenrecipe#NameOpenWindow`
- `internal/battencli#newFakeReceiver`

**Intent:**
D8: Open-Window counts per machine, by the live fact of an attached client.
`battenCLI` gains `clientAttached func() (bool, error)`, which `wire` fills over the task worktree's reed engine, located on call like every other seam.
`battenPreRun`, once a found status passed `reconcileResume`, calls `reopenWindow`: when the status history holds an Open-Window entry and the current producer is past it, and `clientAttached` reports no client, it runs the Open-Window row's command once through a `shedadapters` command producer built over the row's `batten.open_window` switch and `batten.open_window_command` command and `c.env.Command`, so `--window`'s forced switch and a task's `open_window: false` both apply as in the row itself; a client attached opens none; a `clientAttached` error is warned about and opens none; the command's own failure only warns, as in the row.
A first run of the row is unchanged, and a same-machine resume after a detach opens a window again by intent.
`battenVerbTexts`' run help, `battenshed`'s doc and `docs/overview.md`'s batten entry replace "A run resumed past that row opens no window" with the rule above.
`TestBattenPreRun_ReopensTheWindowWithNoClientAttached` is a table over a fake receiver with a recording command runner: past Open-Window with no client runs the command once; with a client attached runs none; with the switch off and no `--open-terminal` runs none; with the switch off and `--open-terminal` runs it once; a `clientAttached` error warns and runs none; a status still at Open-Window runs none from the pre-run; a fresh start runs none from the pre-run.

**ImpactSummary:** batten's resume pre-run gains one Command row call; the recipe and its rows are unchanged.

**Commit:** 20: batten: a resume opens the window on a machine with none attached
