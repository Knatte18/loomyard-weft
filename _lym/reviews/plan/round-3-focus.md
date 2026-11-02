---
round: 3
focus:
  - "Card 9 (`_lym/plan/09-goto-ends-fixers.md`), `endGateFixers`: settle whether any error it returns carries a shed transient mark, and whether `PATTERN-transient-stop` (the mark is `internal/shedengine`'s, translated by `internal/shedtransient` at a producer or step-bootstrap boundary) permits it in a goto guard in `internal/loomcli`; check the Intent and the Tests section agree."
  - "Card 9, ordering: settle whether the Intent's statement of when fixers end matches `internal/shedengine/goto.go`, where `Goto` runs its checks before `checkGuard` and `Relaunch` runs the guard inside the status-lock callback before `req.Admit`; check the refused and already-in-effect cases are stated, and that `GotoGuard`'s doc comment, which says the guard only decides and writes no file, is covered by a target."
  - "Card 9, docs: settle whether `internal/loomcli/doc.go`'s paragraph on the told goto guard's two arms stays true after the wrapper, and whether its Uses or Extends label fits the card's change."
  - "Card 3 (`_lym/plan/03-webster-fixer-on-core.md`): settle whether every comment the Intent rewrites to drop a deleted member name, in `internal/websterengine` files `render.go`, `strand.go`, `recoverbatch.go` and the `testReplyPath` doc comment in `template_test.go`, sits under some card's target, given `redundant-file-target` bars a file glyph beside a member glyph of the same file on one card."
---

Focus targets the sites round 2 touched; every entry is a question, none a verdict.
