```yaml
lens: row-validity
outcome: reported
```

Row-validity review of the four-card plan: whether each planned test row reaches its branch, would fail if the code were wrong, and is needed.

**MEDIUM — Card 4, D3 frame layout: the new section breaks in `frame` are never tested.**
- `TestOverviewSource_DrawsTheSelfRunLine` builds every row with no orch state file and no dobby, so each frame holds the self-run line alone or nothing (card 4 Intent, "so the frame holds the self-run line alone or nothing").
- The card changes `internal/battencli#overviewSource.frame` to draw the dobby lines, then the self-run line, then the orch lines, with a blank line between each two sections present.
- No row reaches the self-run line next to orch lines or dobby lines. A missing or extra blank line, or a wrong order, would pass every row.
- `TestOverviewSource_DrawsTheOrchLine` uses the prime with no seed, so it never reaches the new line either.
- Fix: add a row that writes the beam's orch state through `writeOverviewFile`, as frame 3 of `TestOverviewSource_DrawsTheOrchLine` does, and asserts `[selfRunLine, "", orchLine]`. If the dobby-to-self-run break must be pinned too, add a row with `source.dobby` set to a fixed func.

**LOW — Card 3, `TestStart_EnsuresTheRunsStrandBeforeTheOrch`: the new rows' loom-status guid is not stated, and the shared loop requires `r1`.**
- The loop fails any replace whose guid is not `r1`: `if replaced.replaced != "r1"` in `internal/orchcli/start_test.go` (the spec loop of `TestStart_EnsuresTheRunsStrandBeforeTheOrch`).
- Card 3's new rows say only "each with a live `loom-status` strand". A guid other than `r1` fails every new row that replaces.
- Fix: state `{GUID: "r1", Name: "loom-status", Live: true}` for the new rows, or make the loop compare against a per-row `wantReplacedGUID`.

**LOW — Card 3: the self-status writer the rows need is not named.**
- `selfState` rows write a `shedengine.Status` JSON at `shedrun.StatusFile(h.cli.location, shedrun.SelfRunID)`.
- No orchcli test file holds a status writer: grep for `shedengine.Status{` across `internal/orchcli/*_test.go` finds nothing.
- The battencli `writeOverviewStatus` (`internal/battencli/overview_test.go:308`) is in another package.
- Fix: say the row inlines `json.Marshal` and `os.WriteFile`, or declare a `plan:internal/orchcli#writeSelfStatus` helper under `**Create:**`.

**LOW — Card 1 and card 2: deleting `TestStatusStrandCmd` drops the pwsh command row.**
- `internal/loomcli#TestStatusStrandCmd` covers both `shell.Posix()` and `shell.Pwsh()`, the latter with `C:\lym.exe` (`internal/loomcli/bootstrap_test.go:271-290`).
- Card 1's `StatusStrandAddSpec` hard-wires `shell.ForGOOS()`, and its test asserts only the host shell's command. The pwsh composition goes untested on a POSIX gate.
- Card 2's claim that card 1's `TestStatusStrandAddSpec` "carries" those cases is therefore false for the Pwsh row.
- Fix: keep a shell-parameterised command builder in `internal/loomengine`, for example `plan:internal/loomengine#statusStrandCmd` taking a `shell.Shell`, with the two-shell table moved onto it. Or keep `TestStatusStrandCmd`'s rows against the moved function.

**LOW — Card 2, outside this lens: the `sharedbootstrap_test.go` header goes stale.**
- Card 2 adds `TestEnsureStatusStrandOn` to `sharedbootstrap_test.go`.
- That file's header says ensureStatusStrand's branches are "covered directly through that pure function in bootstrap_test.go rather than re-covered here" (`internal/loomcli/sharedbootstrap_test.go:1-5`). The new test makes that false.
- No card lists the file's header comment.

Tool calls made beyond my inherited context:
- Read `bootstrap_test.go:271-320` (`TestStatusStrandCmd`) and `smoke_drivingsurface_test.go:87-130`: I needed the rows of the test card 2 deletes, and to confirm `loc` is in scope at the smoke test's first `statusStrandAddSpec` call.
- Grepped the orchcli test fakes, the shedengine state constants, `shedrun.WriteSeed`'s body, the battencli overview test helpers, and orchcli tests for a status writer: I needed whether the rows' fixtures can be built with what exists.
