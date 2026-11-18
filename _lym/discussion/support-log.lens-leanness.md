```yaml
lens: leanness
outcome: reported
```

Task: leanness lens review of the Hardener decision record and support log.

1. LOW. In `## Constraints`, the bullets that only restate PATTERN entries (shed-recipe-registry, shed-driver/run-fsm, gate-slot, stencil-ownership, github-auth) repeat compliance that D1, D3, D4 and D11 already cite inline. The plan writer gets the PATTERN index in its own prompt. Keep the bullets that add something specific to this task: lymtemp for the sandbox root, `_lym/harden` through lymdirs, no hubforge or testkit import in production, and the `.dev-bin` lym exception to `selfexec.LymBinary`. Cut the bare restatements.

2. LOW. In `## Constraints`, the three tool-fact bullets (reed `-L` together with tmuxkit, `deploy-dev`, CloneAndWire from local bares) carry design-critical facts with pins, so they stay. The CloneAndWire fact is also stated in D5's rationale ("`internal/hubforge` proves…"). Keep it in one place, Constraints, and point D5 at it.

3. LOW. D6's closing rationale says "this answers the entry's open question of what the handoff must carry…". That is deliberation narrative, and the question ledger already holds it as Q4. The decision content (the list of facts) stays. Move that sentence to the support log.

4. LOW. D8's rationale includes the history of the operator's 2026-10-16 stencil decision. It is useful as evidence that retiring treadle is licensed, but the sentence is long. Trim it to one clause ("the operator's 2026-10-16 decision kept treadle's stencils only until Hardener is built"). The rest is narrative.

5. LOW. In `## Open risks`, the second bullet (the draft names Treadle and D8 departs from it) is a choice the writer made and justified in D8. Under the rubric it is not a parent-only question unless the writer doubts the task's intent. Either drop it, or keep it and phrase it plainly as a question for the parent. As written it is neither a risk nor a question.

6. LOW. `## Notes for the plan writer`: the first bullet (darn and motif/tapestry as templates) duplicates D1's and D3's rationale. Cut it. The second bullet (the `commit_seam` by name) is a useful hint, so keep it.

7. LOW. In the support log, Interview Turn 1 repeats much of `relevant.md` and the decision rationales. That is acceptable, since the support log is only for the review. No change needed.

8. MEDIUM (outside the leanness lens, noted for emphasis). The acceptance bullet for D4 says "never runs tier 3". That is testable, but D4 point 4's "as the rubric says" pushes the tier-3 rule into prose. It is already covered by the `Review:` rubric bullet, so this is consistent. No action needed beyond confirming that the D4 test asserts only the gate's behaviour.

No BLOCKING findings. Every requirement-bearing sentence I checked passes the completeness-before-leanness test: nothing proposed for moving carries a constraint the plan writer needs that isn't stated elsewhere in the record.

Tool calls made: none.
