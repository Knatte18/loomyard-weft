```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the reviewer's proposals the fixer carried out (F1–F4) and find a case where a proposal is wrong or breaks something elsewhere in the plan.

**LOW — card 3, the reworded `testReplyPath` comment bullet (F2's new target)**
- Changed text: "the `testReplyPath` doc comment, which names `RenderVerifyFixColdPrompt`'s `replyPath` parameter and becomes the placeholder reply path the cold fixer's `gatefixer.RenderPrompt` inputs carry".
- Summary: the new wording ties the constant to the cold verify fixer only, but the plan fixer's test passes the same constant too.
- Evidence: `internal/websterengine/template_test.go:976` passes `testReplyPath` as `RenderPlanFixPrompt`'s reply path, and lines 909 and 921 use it in the cold-fixer rows.
  The comment as planned would describe only one of the two prompts that use it.
  The bullet should say the constant is the placeholder reply path the cold fixer prompts are rendered with, the verify fixer's and the plan fixer's.

**F1, F3, F4 and the rest of F2: no case found.**
- F3, `GotoGuard` under Edit with `internal/shedengine/doc.go` under Uses: `doc.go` lines 78–80 never say the guard "only decides" or writes nothing, so the new sentence leaves it true and Uses is the right label.
- F3, the wrapper's reach: loomcli builds a goto guard only at `arm.go:464`, `arbiterconsume.go:28`, `arbiter.go:140` and `drivewiring.go:171`.
  At `drivewiring.go:171` one `gotoGuard` variable feeds both `TacklerDeps.GotoGuard` (line 208) and the `ConsumeTackler` `Guard` (line 281).
  The card already says to wrap only the consume, so wrapping that one variable would be the implementer's error, not the plan's.
- F3, the in-effect claim: every loom goto path, the cobra verb, `driverGoto` and both consumes, goes through the wrapped guard.
  A paused run steps no row, so no row fixer starts after that goto.
- F4: the two-arm paragraph stays true with a sentence added after it, so Extends holds.
- F1: once the transient mark is dropped, `checkGuard`'s "could not be checked … re-run goto" wrap gives the slug error its way forward, and nothing reads a transient mark there.

Tool calls:
- grep of `testReplyPath` uses, `shedengine/doc.go` guard lines and loomcli guard sites: my context did not hold the other test uses, the shedengine package doc or the guard construction sites.
- `sed` of `internal/shedengine/doc.go` lines 76–84: to read the full guard paragraph the grep matched.
