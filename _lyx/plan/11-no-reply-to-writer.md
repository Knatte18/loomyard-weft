# Card 11 — no-reply-to-writer

**Prosa:**
- `contracts/stencils/loom/loom-template-parent-review-brief.md`
- `contracts/stencils/orch/orch-template-role.md`

**Edit:**
- `contracts/stencils#wordingClaims`

**Intent:**
D4: neither the orch nor its reviewer fork replies to the writer.
A reply starts a writer turn the wait loop did not start, which costs a turn and moves the writer off the boundary while a verdict may be pending.

The brief's `## Submit the verdict` section gains one line: the verdict reaches the writer through the gate only, so send the writer no message, neither an acknowledgement nor a reply to its notice.
The orch role stencil's `## Parent-review` section gains the same rule for the orch and its fork.
Each stencil carries the rule for its own reader: the fork reads the brief, and the orch reads its role before forking.

`wordingClaims` gains a row for `loom-template-parent-review-brief.md` over `LoomTemplateParentReviewBrief`, claiming the no-message line in `## Submit the verdict`;
its `orch-template-role.md` row gains a claim on the same rule in `## Parent-review`.
Each claim's why cites D4: a reply starts a writer turn the wait loop did not start.

**ImpactSummary:** two stencils gain one line each and the claims table pins them; no Go behavior changes.
