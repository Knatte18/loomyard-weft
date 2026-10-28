```yaml
lens: interactions
outcome: reported
```

Task: interactions lens over `decision-record.md`. I checked each pair of decisions and each constraint and acceptance criterion against the decisions it touches.

**Pairs checked**
- D1 removing the pair from `review-discussion` × D1's lens prose ("ends in its list", "reports only") × the Constraint "procedure lenses end in their own list": **broken**.
- D1 × D2, the writer-covers-reviewer test: held. `review-discussion`'s four lenses all stay in `twill-discussion`, and `repo-invariants` is the one reviewer-only lens, which the D2 acceptance line states.
- D1 × the D1 acceptance line "`discussion` names `board-scope`": **broken**, the same defect as the first pair, since `discussion` is no `review-*` or `fix-*` fan.
- D3 × D4 (both rewrite `review-code` lenses): held. D3 moves only the repository-rules bullets and D4 drops only "Test economy", so neither undoes the other.
- D3 × the `csharp` layer fan × the fan-length test: held. The decision states the new four-lens `csharp` fan.
- D5 × D6 (D6's profile blocks need D5's reclassification): held. A `""` in the `light` and `docs` blocks passes `checkLocalDocument` once the key is `CategoryNone`.
- D5 × D6's acceptance line "every block passes configreg's check": held.
- D5 × D7 × the `PATTERN-config-layers` Constraint: held. The background's Guard bullet changes in the same commit, and the overview line names no guard keys.
- D6 × the Constraint "D6's value survives reconcile": held. `ReconcileAll` passes a nil strip to the local file.
- D6 × the Constraint "`profiles.yaml` is seed-only" × the first Open risk: held. Both state that the template change reaches only a newly seeded hub.
- D1 and D3 deleting lenses × the second Open risk (a hub fan naming a deleted lens): held as a pair. The breadth of that risk's claim is outside this lens.
- D2 × the `validator-checks` Constraint: held.
- D4 × the unchanged code rubric (the review lead still judges test economy): held.
- D5's bound "`code_max_bounces: 0` is the stronger switch" × D6: held.

**Findings**

1. MEDIUM, `design`, D1 (and the Constraint on lens prose, and the D1 acceptance lines). D1 takes `board-scope` out of every `review-*` fan, which strips its procedure-lens status, yet its prose ends in a list and "reports only" contradictions.
   - The template header defines procedure lenses as those of "each review-* and fix-* fan" (`internal/lenses/template.yaml:33-36`).
   - `TestConfigTemplate` builds `procedureLenses` from `review-`/`fix-` fans only, and requires every other lens to end with "Report anything else you notice too — emphasis, never exclusion." (`internal/lenses/config_test.go:296-300`, `:341-343`).
   - `goal-scope` and `board-fit` are exempt today only because they sit in `review-discussion`. `board-scope` sits only in `twill-discussion` and the dormant `discussion` fan, so the test demands the emphasis line.
   - That line contradicts D1's "Reports only a contradiction with the board" and its acceptance line "reports only contradictions with the board". The Constraint "procedure lenses end in their own list" does not cover a lens that is not a procedure lens by the template's own definition.
   - The record never says which side gives: the emphasis ending, or a change to how the template and test define a procedure lens.
   - Evidence: `internal/lenses/config_test.go:296-300, 341-343`; `internal/lenses/template.yaml:33-36, 390-410`; decision-record D1, Constraints, Acceptance criteria D1.

Tool calls: none.
