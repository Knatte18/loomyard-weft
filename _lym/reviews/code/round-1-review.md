---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    origin: lens:code-affected-set
    location: contracts/stencils/harden/harden-rubric-review.md:98
    summary: The judge rules CONVERGED only when the round's reviewer model differs from the previous round's, so a campaign on the new one-model default can never converge and always runs to its round cap.
    fix: "In contracts/stencils/harden/harden-rubric-review.md, under `## The judge`, the **Converged.** bullet: drop the clause `, and whose reviewer model differs from the previous round's` so the rule reads `The judge rules CONVERGED only on a round whose review records no BLOCKING or MEDIUM finding and whose live gate is green.`, and change the next line from `It reads all three from the round's facts.` to `It reads both from the round's facts.` Under a rotation the clause already always held, since CheckCampaign refuses a list of two or more entries that puts one model in neighbouring rounds, so dropping it loses no check. Leave the Go code untouched."
---

The diff makes every change the amended entry asks for except one.
The template sets `reviewer` and `fixer` to `["sonnet[high]"]`, and the template comments and the `hardenshed` package doc state the one-model default and the per-run rotation.
A new `TestPrepareRunSettings` row shows that `run_settings` takes both keys as lists, and no code fix was needed.
Preflight now passes a one-entry reviewer list and still refuses neighbouring equal models in lists of two or more.
Every lens traced `reviewerRepeat` and `CheckCampaign` at lengths zero, one, two and three, and the refusal is correct at each.
Each changed behaviour has a test row that would fail if it broke.
No assertion that existed at the review base was loosened.
The one assertion removed went with the refusal that the parent's added entry withdraws.

What the diff misses is the hardener rubric's convergence rule, which still assumes rotation.

### [BLOCKING] The judge's convergence rule still requires a change of reviewer model

**Location:** `contracts/stencils/harden/harden-rubric-review.md:98`

**Issue:** The **Converged.** bullet rules CONVERGED only on a round "whose reviewer model differs from the previous round's", and the judge reads that from the round's facts.
`internal/hardenshed/campaign_rows.go:186` writes the facts' `Models:` line from `Rotate` over the reviewer list.
With the new default, `internal/hardenshed/template.yaml:5`, every round's reviewer is `sonnet[high]`.
A campaign on the default therefore can never converge: however clean its rounds and however green its live gate, it runs until `max_bounces`.
That breaks the operator's decision that one model with no rotation is the working default.
It also breaks the parent's added entry, whose point is that the default campaign runs.
The goal's third point asks that doc text built on rotation as the default follow the change, and this stencil line is such text.
The scope and affected-set lenses found this independently.
The assistant reviewer and the remaining lenses missed it.

## Rejected

- Affected-set lens F2 (NIT): flagged the `Pattern-Departure: none` trailer on `0196d0f69`. Rejected: `none` declares that the commit departs from no PATTERN entry, it names no unknown entry, and no departure is being excused.
- Code-correctness lens: the refusal message's "its last and first entries included" also prints for a pair that doesn't wrap around. Not raised by that lens, and rejected here too: the text predates the diff and describes what the check covers.
