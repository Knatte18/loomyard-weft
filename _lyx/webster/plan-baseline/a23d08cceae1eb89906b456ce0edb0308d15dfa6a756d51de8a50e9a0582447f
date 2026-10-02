# Card 4 — fork-fixes-own-breakage

**Edit:**
- `contracts/stencils/webster/webster-body-implementer.md`
- `internal/websterengine/template_test.go`

**Intent:**
The fork implementer fixes a failing test its own card caused before it reports (#332a, decision D4).

Rewrite the "Bounded self-fix, then stop" section of `webster-body-implementer.md`:
- Any failing test the fork observes, in its gate or in any other test run it makes, counts as that card's gate failure when the card's change caused it.
- Caused-by-the-card means the test exercises code or files the card changed, or a repo scan or enforcement test flags a file the card wrote.
- The fork fixes such a failure under the same `{{.self_fix_cap}}` bound as any gate failure, and reports `status: FAILED` when the bound runs out.
- A failure the card did not cause is named in the fork's final reply to Master, never fixed, and never a `FAILED` on its own.

Align the sentence above that section ("If any card's gate fails ...") and the report section's `FAILED` definition with the widened gate.
What the fork must run is unchanged: this card adds no test command.
The report's fields stay `status`, `head_sha` and `deviations`.

Add a stencil property test to `template_test.go`, in the style of the master template's: render a fork prompt with `RenderForkPrompt` and a known self-fix cap, and assert that the "Bounded self-fix, then stop" section carries the caused-by-the-card rule (both causes), the rendered cap value, the `FAILED`-on-exhaustion rule, and the report-but-never-fix rule for a failure the card did not cause.

**ImpactSummary:** every fork and recovery prompt (both compose this body) gains the caused-by-the-card rule; no Go behavior changes.
