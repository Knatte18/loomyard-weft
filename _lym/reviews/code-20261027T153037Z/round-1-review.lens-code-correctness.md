```yaml
lens: code-correctness
outcome: reported
```

**Task:** code-correctness lens over the diff's changed functions and branches.

**List: function or branch, case run, result**

- `DriftBaseline.Call`: `Sleep` seam dropped, `commitAll` starts false. Held lock with scope full returns Done and commits only the scope record and `baseline.md`. Correct.
- `drive`, held branch: returns Done with the report pointer before reading the capture record. Correct.
- `drive`, no lock and a passed record: Done, nothing started. Correct.
- `captureAgain`, `StartCapture` nil: Done, `commitAll` false. Correct.
- `captureAgain`, `ErrBreakawayRefused`: `commitAll` true, `RunCapture` in-process; passed gives Done, failed gives Stuck with `CaptureHaltPrefix`, a `RunCapture` error is returned with `commitAll` true. Correct.
- `captureAgain`, any other start error: returned wrapped, `commitAll` false. Correct.
- `captureAgain`, config or `checkCapture` refusal: unchanged, Stuck. Correct.
- `CountedCapture`: absent, running, failed, passed without `awaited_at` false; passed with `awaited_at` true. Correct.
- `Waive` and `DriftVerify.pass`: use the three-argument `CountedCapture`. Correct on every recipe.
- `driftBaselineEntry`: `{"subject":"board-entry","capture":"wait"}` is refused with the `driftKeyWayForward` message. Correct.
- `driftBaselineEntry` seam list: `Drift.Sleep` no longer required for Baseline; the Await entry still requires it. Correct.
- `driftcli` verb status: the seed read is gone; nothing else read the seed. Correct.
- `loomcli.driftDeps` and `wire`: parameter removed from definition and call site. Correct.
- `darnPhases`: `Drift-Await` maps to `PhaseSetup`; without it a darn Drift-Await entry would be an unmapped row and produce a finding. Test arithmetic checks out. Correct.
- `darn-recipe.yaml`: Baseline `on_done` Drift-Await; Drift-Await has `entry_gate: drift.enabled`, `on_done: Darn-Write`, no config or `on_stuck`. Matches `driftAwaitEntry`. Correct.
- `DriftAwait.await` in darn: `capture.Head != head` is the same check loom runs, with the same Baseline commit between capture and Await. I did not re-derive how that comparison tolerates the Baseline commit; loom already depends on it.

**Findings**

1. **LOW, class `consistency`**: the transition note in `contracts/recipes/darn-recipe.yaml:51-52` is narrower than the runs the change affects.
   - The header says a run "parked between Drift-Baseline and Darn-Write" is settled by hand.
   - The old darn wait mode never set `awaited_at`; `DriftAwait` is its only writer.
   - So any in-flight darn run past Drift-Baseline whose capture passed, including one in the long `Darn-Write` row, has no `awaited_at` once the new binary takes over. `CountedCapture` returns false, and it halts at Drift-Verify with "no baseline capture counts".
   - `lym drift waive` settles it by hand, but the header understates the window.
   - The behaviour matches the operator decision of no transition code. The finding concerns only the wording of the header comment.

No other correctness defects found.

**Tool calls (1)**
- Read `internal/driftengine/await.go` lines 56-120, to confirm the head and `awaited_at` logic Drift-Await runs for darn.
