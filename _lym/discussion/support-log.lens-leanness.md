```yaml
lens: leanness
outcome: reported
```

Task: review the decision record and support log through the leanness lens, looking for content to move or cut.

**Findings**

1. LOW — Auto-mode assumptions. Each of the five bullets restates a decision already in full: the `arras` name (D10), `cartoon` as a block source only (D9), no `on_stuck` on a block row (D4/D1), the block row as its own step with names kept (D7/D2), and `profile_recipe` (D8). This breaks the "say it once" rule. Evidence: compare each bullet with the decision it repeats. Fix: cut each bullet to the pick and a decision id, e.g. "`arras` as the name (D10)".

2. LOW — The `arras` name approval appears three times. It is in D10's last line, in Auto-mode assumptions and in Open risks. Fix: keep the Open risks line and drop the sentence from D10, since its "(Open risks)" pointer already covers it.

3. LOW — Open risks, second bullet (reports grouping runs by row mix tapestry and `arras`). This is advice for the out-of-scope bench task, not a risk to this design. Completeness test: the plan writer can't act on it here. Fix: move it to the support log's question ledger, or to a board note for the bench follow-up.

4. LOW — Constraints, last bullet ("Tapestry's recipe file and every tapestry run in flight are unchanged…"). Goal, Scope, D9 and D10 already say tapestry stays unchanged. Completeness test: the part about row names being durable resume identities is the only new content. Fix: keep just that clause.

5. LOW — D6's last sentence ("An older status file decodes with the field empty.") repeats the `PATTERN-tolerant-records` constraint and D6's own rationale. Fix: cut the sentence. The acceptance criterion "a status file without the field still reads" keeps the requirement.

6. LOW — The rationales of D7 and D11 mostly restate their decision text. Fix: shorten each to one clause, or drop them, since the decision text carries the reason.

7. LOW — D3, "Validation refuses a skip target that names no producer". This is plan-level detail of how `shedengine` validates. Completeness test: it is an invariant the plan needs, so keep it. It is borderline only.

8. MEDIUM (noticed outside the lens: consistency) — D2 and D8 disagree. D2 refuses a sub-recipe that sets `contract`, `verify_source` or `amendment_target` because these "belong to the run, not to a block". D8 adds another run-level top-level key, `profile_recipe`, but D2's refusal list doesn't include it. So a block recipe that sets `profile_recipe` would be silently ignored. Evidence: D2's refusal list compared with D8's new key. Fix: add `profile_recipe` to D2's refusal list and to its acceptance criterion.

9. LOW (noticed: emphasis) — The Notes for the plan writer list call sites. The rubric says not to flag incomplete enumeration, and the closing "grep for `RecipeTapestry`" already hands the search to Plan. No change needed; it's noted only because the list could go stale.

**Tool calls**

None. My context already held the record, the support log, the rubric and the exploration.
