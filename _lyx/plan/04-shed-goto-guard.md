# Card 4 — shed-goto-guard

**Edit:**
- `internal/shedengine#GotoRequest`
- `internal/shedengine#RelaunchRequest`
- `internal/shedengine#Goto`
- `internal/shedengine#Relaunch`
- `internal/shedengine#TestRelaunch`
- `internal/shedengine/doc.go`

**Create:**
- `internal/shedengine#GotoGuard`
- `internal/shedengine#checkGuard`
- `internal/shedengine#TestGoto_Guard`

**Uses:**
- `internal/shedengine#gotoShed`
- `internal/shedengine#gotoRequest`
- `internal/shedengine#seedWith`
- `internal/shedengine#readStatus`

**Intent:** The D2 seam in Shed. Shed knows no review state, so the recipe owner tells a `GotoGuard` on `GotoRequest.Guard` and `RelaunchRequest.Guard`; nil admits every target, as today, and batten tells none.

`checkGuard` runs the guard for a target: a non-empty refusal returns `shedengine: goto target "<row>" refused: <refusal>`, the refusal text carrying the recipe's own way forward; a guard error returns `shedengine: goto target "<row>" could not be checked: <err>; way forward: re-run goto` (naming `runVerb`-style addressing with the run-id), so a seam error refuses too.
`Goto` calls it after `CheckGotoTarget` and before the write, under the run lock, so every refusal leaves the status file unchanged.
`Relaunch` calls it for a non-empty target after `CheckGotoTarget` and before `Admit`, so a refused target never lets `Admit` write its applied mark.
Bound: the guard only refuses; it moves no run and writes no file, and Shed never consults it for a target its own checks already refused.

`doc.go`'s goto paragraph names the told guard and its refusal.

`TestGoto_Guard` is one table over `gotoShed`: a guard refusal returns an error holding the refusal text and the file is unchanged; a guard error refuses with `re-run goto` and the file is unchanged; an admitting guard moves the run; a guard is not called for a target past the current row.
`TestRelaunch` gains two rows: a guard refusal and a guard error each return their refusal, call `Admit` zero times and leave the file unchanged.

**ImpactSummary:** Two request structs gain an optional field; every existing caller leaves it nil and behaves as before.

**Commit:** 4: shed: goto and relaunch consult a told guard before moving a run
