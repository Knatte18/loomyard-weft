---
round: 1
focus:
  - "internal/loomcli/wiring.go `wire`: the task adds `VerifyHold: verifytree.NewPauseHold(pauseGeom, nil)` while the parent removed `c.env.StencilsDir` from `wireLightweight`; does the merged `wire` still build the same `shedrecipe.Env` the seam test `wiring_seams_test.go` and the parent's lightweight wiring assume, and is `pauseGeom` in scope and set on every path that reaches that line?"
  - "internal/websterengine/doc.go: the task's verify-gate paragraph (`Geometry.VerifyHold` on the first run and the rerun) against the parent's rewritten post-batch pass and its removal of the kick-start pack sections; does any surviving doc sentence still describe pack or `card-pack` behavior the parent deleted, or does the task's paragraph depend on text the parent moved?"
  - "internal/websterengine verify gate (`verifygate.go`) against the parent's changes to `recordbatch.go`, `config.go` and `render.go`: does anything the parent changed alter how the gate closure, `Geometry`, or the recovery path reaches the told `VerifyHold`, so that a pause no longer holds the gate's run and rerun?"
  - "PATTERN.md and the changed `pattern/` files (`PATTERN-glyph-conversion-chokepoint.md`, `PATTERN-review-round.md`): does the task's pause-hold code (verifytree `PauseHold`, its callers in landingshed, websterengine and loomcli) follow every invariant the parent added or changed there, and does the parent's new code break a pause-hold invariant the task relies on?"
  - "Test economy: tests the parent removed or rewrote in `internal/websterengine` and `internal/loomcli` against the task's new hold tests; is any pause-hold interaction (gate first run, rerun, nil hold) left without a covering test, or is a task test now redundant with a parent one?"
---

Seed focus from the overlapping files and packages only; the reviewer reads the full merge diff.
