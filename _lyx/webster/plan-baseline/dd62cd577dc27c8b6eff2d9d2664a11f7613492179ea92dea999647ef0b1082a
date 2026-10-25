# Card 30 — batten-continue-pass

**Edit:**
- `internal/battencli#battenCLI`
- `internal/battencli#Command`
- `internal/battencli#battenCLI.battenPreRun`
- `plan:internal/battencli#passExitCode`
- `plan:internal/battencli#TestPassExitCode`
- `internal/battencli#TestBattenPreRun_ReopensTheWindowWithNoClientAttached`
- `PATTERN.md`
- `pattern/PATTERN-owner-machine.md`

**Create:**
- `plan:internal/battencli#continueRefusal` -> `type continueRefusal struct`
- `plan:internal/battencli#battenCLI.refuseContinuePass` -> `func (c *battenCLI) refuseContinuePass() error`
- `plan:internal/battencli#TestBattenPreRun_ContinuePass` -> `func TestBattenPreRun_ContinuePass(t *testing.T)`

**Uses:**
- `plan:internal/battencli#OriginOwed`
- `plan:internal/hubgeom#DobbyGeometry`
- `internal/dobbyengine#ReadState`
- `internal/dobbyengine#PassExitContinueRefused`

**Intent:**
Add the continue pass dobby schedules into a run it already drives, and the full pass that skips only the window re-run (D4).
The run verb gains two hidden flags: `--continue-pass`, which skips the resume's origin reconcile (`preflightRunOwner`), the Open-Window re-run (`reopenWindow`) and the pre-run pause refusal (`refuseCovered`), and `--no-reopen`, which skips only the Open-Window re-run, so a run whose origin-owed marker stands opens no window per probe tick.
Bound: `refuseContinuePass` runs first under `--continue-pass` and refuses with a `continueRefusal`, naming `lyx batten run <slug>` as the way forward, unless batten's status reads `running` at Run-Shed, the state record of the parent worktree's dobby (`hubgeom.DobbyGeometry` of the run's parent, `dobbyengine.ReadState`) names the run as one it drives with no full pass owed, and `OriginOwed` reads no marker; `passExitCode` maps it to `dobbyengine.PassExitContinueRefused`, which dobby reads as a full pass owed.
The flag skips nothing else: the owner check's pause-lift and owned-elsewhere triggers run inside the pass as before, and a continue pass under a covering pause runs its one check, which keeps releasing an idle owner's parent lock and sending the child's notices.
A hand pass under a covering pause keeps today's refusal naming `lyx resume`.
`PATTERN.md`'s owner-machine entry becomes:

```
- `PATTERN-owner-machine` — a continue pass dobby schedules into a run it already drives is not a resume and needs no origin verdict; the first pass of a run after a dobby start or a machine wake, every pass while a full pass is owed, and every hand pass, is a resume and gets one. — [background](pattern/PATTERN-owner-machine.md)
```

and the background file gains a "Passes" section stating it, beside its origin-reconcile-first rule, which every full pass keeps.
Tests: `TestBattenPreRun_ContinuePass` (tier0, parallel) over a hand-populated receiver and a temp dobby state: a continue pass of a running Run-Shed run the state names runs no reconcile, no reopen and no pause refusal under a covering pause; a run at Land, a run blocked at Run-Shed, a run the state does not name, a run with a full pass owed and a run with an origin-owed marker each refuse naming `lyx batten run <slug>`; `TestPassExitCode` gains the continue row (79); `TestBattenPreRun_ReopensTheWindowWithNoClientAttached` gains a `--no-reopen` row that runs the reconcile and opens no window.

**ImpactSummary:** Two hidden flags on the run verb, gated by one refusal read off the parent dobby's state and the marker; every pass without them runs today's full pre-run.
