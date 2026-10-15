# Card 4 — Verify-gate fixer: the inherited context holds no code

**Edit:**
- `contracts/stencils/webster/webster-body-verify-fix.md`
- `contracts/stencils#wordingClaims`
- `internal/websterengine_test#TestRenderPrompts`

**Intent:** Carry decision D5.
In `webster-body-verify-fix.md`, "You inherit Merriam's whole context, so this prompt is deliberately thin." stays, and the line after it adds: "That context holds no code: Merriam never reads it, so read the code behind each finding yourself before you edit it."
Every other line and every marker is unchanged.

In `contracts/stencils#wordingClaims`, the `webster-body-verify-fix.md` row gains a `wantAll` of "That context holds no code" and "read the code behind each finding yourself".

In `internal/websterengine_test#TestRenderPrompts`, the row "fixer prompt names the gate report and the contract files" gains `contains` "That context holds no code".
This render needle is kept on purpose beside the claim row: acceptance criterion 6 names the rendered verify-fix prompt.

**ImpactSummary:** One stencil, its claim row and one render row; no marker or Go code changes.

**Commit:** 4: webster: the verify-gate fixer reads the code behind each finding itself
