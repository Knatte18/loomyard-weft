---
verdict: APPROVED
---

No findings.

The range is `git diff $(git merge-base main HEAD)..HEAD`: no rework round exists, and `_lyx/shed/run-window-bugs/status.json` names `main` as `product.parent`.

Each focus question is settled below.

- Cards 1 and 2, `internal/shuttleengine`:
  - `Run.loadSkillsThenPrompt` awaits the launch-carried first turn through `loadSkillTurn(..., typed=false)` before `typeStartupColor`, the typed retry turn and the prompt.
  - A timed-out first turn warns and continues; a pane that dies during it ends the startup died through `abandonStartup`, and no color is typed.
  - `turnStartedAfterSessionStart` reads the events through the engine's `SessionSignalParser`, never by hook name, and `checkLivenessTick` calls it only with `eventsProof` set: `awaitStartup` passes true and `Run.Wait` passes false.
  - It runs after the reed-status, liveness and `sessionGone` checks and before `CapturePane`; a false answer falls through to the capture path.
  - Every new wait runs on `run.clock`.
- Card 3, `internal/statuspane`:
  - `redraw` writes cursor home, erase screen and erase history before the lines.
  - `Run` selects on `ctx.Done()`, `After(wait)` and `Resized`, so the idle back-off is unchanged.
  - The tests use a fake `After` and a buffered `Resized`, with no real-time wait.
  - Both panes, `checklistCmd` and `overviewCmd`, switched from `Sleep` to `Resized`.
- Card 5, `tmuxkit`: `BindStaleSocket` moved into `stalesocket.go`, `bindStaleUnixSocket` is gone from `reedengine`, and all three callers use the kit helper.
- Card 6, `lyx loom fmt`:
  - `planGoFormat` checks every named file before any write, and the verb formats only `.go` files inside the worktree.
  - `fmt` is in `verbUsesLightweightWiring` and `resolveRunID`'s non-run-id verbs, the refusal rows are in `refusal-spec.md`, and the edit directive names the verb.
  - `TestFmtCmd` is the one new top-level test, matching the `internal/loomcli` budget row's single increment.
- Card 7, `internal/fabricengine`:
  - `removeWorktreeLeftover` lives in `destroy.go`, retries at most three attempts with sleeps of one then two seconds through `seam.sleep`, and retries only an `isLateWriterError`.
  - `TestRemoveWarpWorktreeDir_LeftoverRetry` is the one new top-level test, matching the `internal/fabricengine` budget row's single increment.
  - `PATTERN-fabric-destruction-chokepoint` and `doc.go` name the exception.
- Docs: the `shuttleengine` and `claudeengine` `doc.go` paragraphs, `PATTERN.md`, `docs/overview.md` and the `loomcli` `doc.go` match the code as landed.
- Card 4 is a doc comment on `PushWithRetry`, and it follows the surrounding comment conventions.

Test runs, all green:

- `lyx gate test` over the touched `internal/*` packages, `contracts/stencils` and `cmd/lyx` (the budget and scan tests).
- `lyx gate test --tags integration ./internal/fabricengine`.
