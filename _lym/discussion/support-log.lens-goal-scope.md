```yaml
lens: goal-scope
outcome: reported
```

Task: review the decision record's fit with the `review-setup-tuning` board entry (goal-scope lens).

**Findings**

1. **MEDIUM — D1 (`review-discussion` part), class `scope`.** The record adds `board-scope` to `review-discussion`, which neither the entry nor the audit asks for.
   - The entry's lens line covers only `twill-discussion` ("merge `goal-scope` and `board-fit` into one board-and-scope lens … keep the other five").
   - The audit says of `review-discussion`: "Recommendation: keep the four". Deleting the two lenses therefore points to a four-lens `review-discussion`, not a fifth lens in that fan.
   - The support log's rejected alternative says "the entry asks for a merge", but the entry asks for it only in `twill-discussion`.
   - The writer can settle this. Either drop the pair from `review-discussion` and leave four lenses, or state why a fifth reviewer fork is worth its cost. The writer-covers-reviewer test holds either way.

2. **MEDIUM — Open risks / D6, class `decision`.** Question for the run's parent: does the entry's profile line ("the Code review is a solo reviewer in the `light` and `docs` profiles") mean only the `profiles.yaml` template, or also this hub's live, seed-only profiles?
   - The record changes only the template and leaves the live file to the operator after deploy.
   - On this hub the live `docs` block has `code_max_bounces: 0`, so the stated outcome does not happen here without further operator action.
   - If the parent meant the live hub, the follow-up needs a named owner, for example a board note, not just an Open-risks line.

3. **LOW — D5, class `scope`.** Reclassifying `code_reviewer_fan` from a guard key to an open one is work the entry does not name.
   - It is a forced consequence of the profile line (`checkLocalDocument` → `ValidateLocal` refuses guard keys).
   - It is bounded and recorded with its `PATTERN-config-layers` background change.
   - No change needed. It is listed so the parent sees that a guard opens under this entry's license.

**Goal points of the entry**

- `twill-discussion`: merge `goal-scope` and `board-fit` into one lens that reports only contradictions with the board; keep the other five. Met by D1.
- `review-plan`: replace `way-forward` with a repository-invariants lens covering test seams and tier tags, `doc.go` and PATTERN text, and card order; keep `guard-reach` and `affected-set`. Met by D2.
- `review-code`: fold `code-pattern-fit` into `code-affected-set`. Met by D3, with the csharp layer included.
- `review-code`: narrow `code-tests` to missing coverage of changed behaviour. Met by D4.
- Profiles: a solo Code review in `light` and `docs`, with `solid` and `wide` keeping the fan. Met in the template by D5 and D6; the live hub file is subject to finding 2.
- Out of scope: delta review, round counts per profile, and new validator checks. All respected, with none deferred into this task and none done.
- "Lens stencils under `contracts/stencils`": the record correctly finds no lens names there (Constraints and Turn 2). Not a gap.

**Decision notes:** the board list shows no `Decision for review-setup-tuning:` note, so there were none to read or that I failed to reach.

**Tool calls:** none. My context already held `lym board list`, `lym board get` for this task, and the audit report from the writer's exploration.
