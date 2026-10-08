---
round: 3
focus:
  - "Open risks in `_lyx/discussion/decision-record.md` says the Plan-Review rubric is the backstop for an unmarked signature change, while D6 gives that rubric only a finding for an `Edit` card holding Go file paths and naming symbols in its Intent. Settle whether some Decision builds a rubric finding for an `Edit` card that changes a member's signature without a re-sign arrow, or whether the risk is worded as having no mechanical or rubric backstop; check `contracts/stencils/loom/loom-rubric-plan-review.md` for what it holds today."
  - "D3 and its acceptance criteria in `_lyx/discussion/decision-record.md` describe the moment a handle becomes a member glyph. In `internal/planglyph/handle.go`, `CanonicalizeHandles` rewrites a draft handle to `plan:<glyph>` and `planparser.IsHandleRef` still reports it as a handle; `BindHandles` at record-batch makes it a plain glyph. Settle whether the text and each criterion name the transition that actually changes the verdict, and which unit a `Rename` to-side handle counts under."
  - "D4, D6 and D7 in `_lyx/discussion/decision-record.md` each mention the stencil sentence requiring every handle to be used. Settle that exactly one card is assigned its removal."
  - "The re-sign arrow in D5: settle whether the record states which tier runs the head-to-member match, whether `ValidateDispatch` runs it, and what the arrow does on a `plan:` handle `Edit` target."
---

Round 2 left two MEDIUM design findings and four LOW ones.
Entries name where to look and which question to settle; the reviewer judges severity.
