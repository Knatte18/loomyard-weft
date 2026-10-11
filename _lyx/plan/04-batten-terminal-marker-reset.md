# Card 4 — batten-terminal-marker-reset

**Rename:**
- `internal/battenshed#terminalOpenedFile` -> `plan:internal/battenshed#TerminalOpenedFile`

**Create:**
- `plan:internal/battenshed#ClearTerminalOpened` -> `func ClearTerminalOpened(scratchDir, producer string) error`

**Edit:**
- `internal/battenshed#innerRunProducer.Call`
- `internal/battenshed#innerRunProducer.openTerminalOnce`
- `internal/battenshed/doc.go`
- `internal/battenshed#TestInnerRun_OpensTerminalOnceAfterSpawn`
- `internal/battenshed#TestInnerRun_TerminalOpenGatedPerRun`
- `internal/battenshed#TestInnerRun_TerminalOpenErrorIsWarnedNotEscalated`
- `internal/battencli#battenCLI.battenPreRun`
- `internal/battencli#battenCLI.battenPreStepUnmarked`
- `internal/battencli#TestBattenPreStep_SeedsAbsentStatus`
- `internal/battencli#TestRunCmd_ResumeDispositions`
- `docs/overview.md`

**Uses:**
- `internal/battenshed#terminalOpenedFileSuffix`
- `internal/battenshed#SpawnConfirmedFile`
- `internal/battenrecipe#NameRunShed`

**Intent:** Decision D4.
The terminal window must open once per batten run, and a child whose status file vanishes mid-run and is re-spawned must not open a second one.
In `innerRunProducer.Call`, delete the block that removes the terminal-opened marker when no child status file is found (`if !found { os.Remove(terminalOpenedFile(...)) }`), so a re-spawn keeps the marker; update the `Call` doc comment's sentence about the once-marker so it says the marker is cleared by batten when it seeds a fresh batten status, never here.
Export the marker path as `TerminalOpenedFile` through the Rename pair, for the same reason `SpawnConfirmedFile` is exported: the battencli tests below need the path to plant and check the marker; retarget `openTerminalOnce`, `Call` and the three terminal tests to the new name.
Add `ClearTerminalOpened(scratchDir, producer)` beside the marker's declaration: it removes the marker file and returns nil when it is absent, so the marker name keeps one declarer in this package.
In `battenCLI.battenPreRun` and `battenCLI.battenPreStepUnmarked`, at the one place each seeds a fresh batten status (the `state.UpdateJSON` call reached only when no status was found), call `battenshed.ClearTerminalOpened(c.env.ScratchDir, battenrecipe.NameRunShed)` before seeding, returning its error as the hook's error (`shedverbs.KindUnseeded` in the pre-step, like a failed seed); `c.env.ScratchDir` is the run's scratch directory wiring sets to `BattenDir(location, slug)`, the directory the Run-Shed row's producer writes the marker under, and `NameRunShed` is that row's producer name the marker filename joins.
A resumed run (status found) never clears it.
Bound: the clear runs only where batten seeds a fresh status, so it clears the marker of a window still open only when the durable batten status vanished mid-run while the ephemeral marker survived (risk R4).
Tests, all extending existing ones: in `TestInnerRun_TerminalOpenGatedPerRun`, the `StaleMarkerDoesNotSuppressFreshChildOpen` row becomes `MarkerSuppressesReSpawnOpen` with marker present, no status, and `wantOpens: 0` (one spawn still happens); `TestInnerRun_OpensTerminalOnceAfterSpawn` additionally asserts the marker still exists after the second Call.
In `TestBattenPreStep_SeedsAbsentStatus`, plant the marker at `battenshed.TerminalOpenedFile(c.env.ScratchDir, battenrecipe.NameRunShed)` before the hook and assert it is gone after; in `TestRunCmd_ResumeDispositions`, plant the same marker in every row and assert it is gone for the `absent_status_file` row and still present for every resume row.
Docs: in `internal/battenshed/doc.go`'s `--window` paragraph, state that the once-marker is reset by batten when it seeds a fresh batten status, so a resume keeps it and a re-spawned child opens no second window; in `docs/overview.md`'s batten paragraph, reword the sentence "a once-marker in its scratch directory keeps a re-spawn or a re-run of `lyx batten run` from opening a second one" to say the marker is reset only when a fresh batten status is seeded, so a resume and a re-spawn open none.

**ImpactSummary:** Two battenshed symbols and both battencli seeding hooks change; the marker's readers are all in battenshed, and callers of the renamed path function are three tests and two methods in the same package.
