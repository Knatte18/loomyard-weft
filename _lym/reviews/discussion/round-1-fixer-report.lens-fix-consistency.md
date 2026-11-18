```yaml
lens: fix-consistency
outcome: reported
```

Task: check through the fix-consistency lens that the round-1 change set agrees with its sibling decisions, criteria and cross-references.

**[MEDIUM] The cap on in-place fixes is claimed for every episode, but the fix-here fallback breaks it**
- Changed text: the D4 Bound bullet "one episode fixes at most `pr_review_small_max` findings in place before the verb sends back", and the matching D2 Bound clause "D4 caps PR-Review's in-place fixes at `pr_review_small_max` per episode".
- Summary: in the fallback cases the D4 fix-here list adds, the verb never sends back, so no cap applies there. Those cases are a target entry gate that does not hold and a Code-Review budget of 0 that stamps `zero_rounds`.
- Evidence: D4 says "Fix here ... also when ... the edge's target row's entry gate does not hold, or when the target perch approves without review". Both cases happen inside the tapestry, darn and sampler recipes, which have a Code-Review segment, so the D2 Bound relies on a cap that the D4 fallback removes in those runs.

**[MEDIUM] The D4 wake both stops the session and keeps it up on a dirty tree**
- Changed text: "It stops the session first and re-reads the tree, as it does for the continue flag. On a tree dirty before or after the stop it keeps waiting, with the session up for the agent to commit or discard".
- Summary: after the stop, no session is running. D4 never says the session is started again, so "with the session up" contradicts the stop just before it.
- Evidence: the record already says the wake acts "only on a clean tree", so it never stops on a tree that is dirty before the stop. Only the after-stop case is reachable, and there the stop has already ended the session.

**[LOW] The D6 rubric criterion was not updated for the new review-input channel**
- Changed text: the D6 sentence "The rubric's `## Review input` section, which today reads an input only when the amendment note names one, reads each input the review-input note (D4) names."
- Summary: the twin acceptance criterion still says only "Review: the rubric's `## Review input` section says to review the whole range and file each PR finding that holds (D6)". It does not check that the section now reads inputs from the review-input note.
- Evidence: the record's acceptance criteria list, last Review bullet but one.

**[LOW] D6 leaves it open which note names the Parent-Align fixer report on a plain amendment**
- Changed text: the same D6 sentence, together with the D4 bullet "the reviewer's round prompt names each input in a review-input note of its own, outside the amendment note".
- Summary: today the amendment note names the fixer report on an amendment (`internal/shedadapters/amendment.go:120-121`). The D6 rewrite makes the rubric read inputs from the review-input note, but neither D4 nor D6 says whether the amendment note keeps naming the report or the review-input note takes it over.

I made no further tool calls.
