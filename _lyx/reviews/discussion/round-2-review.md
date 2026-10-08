---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md (D5, D6, D1 Bound)
    summary: After a mid-generation switch the judge can exclude lenses that never ran, since the settled test is vacuously met for a lens with no findings, which D5's own round-1 rationale rules out
  - id: F2
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md (D3)
    summary: The discussion fan emphasizes only gap-finding, with no lens on the rubric's relocation and exclusion item, and the acceptance lens adds a per-decision criterion found in neither the rubric nor the writer's contract
  - id: F3
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md (D3 and D4)
    summary: D3's claim that a rubric change needs no lens edit holds only for a rule change; a renamed or removed rubric item leaves a lens in a seeded, seed-only burler.yaml naming an item that no longer exists
---

### [MEDIUM] Exclusions can name lenses that never ran after a mid-generation fan switch

**Location:** `_lyx/discussion/decision-record.md`, D5, D6 and D1's Bound.

**Issue:** D1's Bound lets a changed `discussion_fan`/`plan_fan` switch a segment between solo and fanned, or between fans, mid-generation.
D5 defines the arithmetic for every switch case: the union is restricted to round N's fan, foreign names are ignored, and the fallback falls back to the whole current fan.
That part holds for round N-1 solo, for round N-1 under a different fan, and for a union naming lenses the current fan lacks.
The gap is in what the judge is told to decide.
D5 tells the judge to exclude a lens when "the latest round's findings from it carry nothing at MEDIUM or worse and its area is not in the next round's `focus`".
D6 gives the judge "the lenses the latest round ran" and per-origin counts.
When the latest round was solo, or ran a different fan, no current lens ran, so every current lens has zero findings and meets the settled test vacuously.
A judge following the rule can then exclude all but one lens of a fan that has never run in this generation, and D5 makes that exclusion permanent until a new generation.
D5's own rationale for round 1 ("no round has run for it to judge settled") is the same situation, but the record applies it only to round 1, not to the first round under a new fan.

**Fix:** In D5, state that a lens is eligible for exclusion only if it ran in the latest round, and that Go drops (with a warning naming them) any `exclude_lenses` entry for a lens the latest round did not run, so the first round under a newly switched-in fan runs it whole.
Add the case to the exclusion Acceptance criterion.

### [MEDIUM] The discussion fan carries the additive bias the rubric is written against

**Location:** `_lyx/discussion/decision-record.md`, D3 (lens list and Rationale); `support-log.md` Interview item 5.

**Issue:** D3's Rationale and the support log say the lenses are drawn from the rubrics' "Also flag" lists.
For the plan fan that is true: `decision-coverage`, `card-structure` and `verify-tests` name Plan-Review rubric items.
For the discussion fan it is only partly true.
The Discussion-Review rubric's Also flag items are relocation and exclusion findings, the completeness-before-leanness test, the attack-surface question and the writer/reviewer symmetry note.
Only `edge-cases` names one of them (attack surface).
`goal-scope` ("nothing the entry asks for missing"), `pattern-fit` and `acceptance` all hunt for gaps, and no lens emphasizes relocation or exclusion, which the rubric singles out as "on equal footing with gap-filling findings" and as the remedy for discussion files that only grow across rounds.
`acceptance` also asks that "every decision has an acceptance criterion that would catch its regression".
Neither the rubric nor the writer's stencil requires that; the stencil only says must-cover scenarios go under Acceptance criteria.
So that lens adds a completeness criterion of its own, which contradicts D3's claim that no lens restates or overrides the rubric and that the rubric stays the authority.
Fanning five gap-seeking forks over a design record multiplies the bias the rubric's relocation item exists to counter.

**Fix:** Give the discussion fan a lens that emphasizes the rubric's relocation and exclusion item and the completeness-before-leanness test by name, replacing or merging one of the gap-seeking lenses so the fan stays the same size.
Rephrase `acceptance` to emphasize what the writer's contract asks (must-cover scenarios are observable and catch a regression of the decision they cover) without adding a one-criterion-per-decision rule, or name the rubric or stencil rule it emphasizes.
Correct D3's Rationale to say which lenses come from the rubric and which from the writer's interview categories.

### [LOW] "A rubric change needs no lens edit" holds only for rule changes

**Location:** `_lyx/discussion/decision-record.md`, D3 (second paragraph) and D4.

**Issue:** D3 says a lens names a rubric item and never paraphrases its rule, "so the segment rubric stays the authority and a rubric change needs no lens edit".
The lens texts live in `burler.yaml`, which D4 keeps seed-only, and D4 uses an operator-file name verbatim.
Every hub seeded after this task gets the `discussion` and `plan` lenses copied into its own file, frozen there.
A rubric change that renames or removes an item (`card-structure` names six Plan-Review items by their headings) leaves those seeded lenses pointing at an item that no longer exists, and nothing reconciles them.
The claim is true for an edit to an item's rule and false for an edit to the item set.

**Fix:** Narrow the claim in D3 to rule changes, and state what a renamed or removed rubric item requires: an edit to the template lenses, which seeded hubs pick up only if their file does not define the name.

## Focus verdicts

- D5 union and fallback over the switch cases: the arithmetic is defined for all three named cases (round N-1 solo contributes no union entries the current fan accepts; a different fan's names are ignored by the restriction; the fallback restricts by subtracting from the current fan, and the whole current fan backs it).
  The gap is in the judge's eligibility rule, raised as F1.
- Seed call and the first Acceptance criterion: D5 states that the seed call gets the focus-only schema, is asked for no `exclude_lenses`, and that a round-1 focus file's excludes are dropped with a warning.
  The first Acceptance criterion names the judge calls as the ones asking for `exclude_lenses` and the seed calls as not.
  Not a defect.
- D3's no-restate claim against the lens texts: raised as F2 and F3.
- The Constraints line on fixer start: it holds for both modes.
  `internal/burlerengine/doc.go` states that under `parallel` the fixer starts at once and waits for the ready marker, and under `after-review` it starts only after the marker is written, and that `reviewReady` writes the marker only after the fork audit (cluster rounds) and the strict parse.
  "Acts only after the ready marker" is true in both.
  Not a defect.
