```yaml
lens: leanness
outcome: reported
```

Task: review the decision record and support log through the leanness lens, looking for content to cut or move.

**MEDIUM — Open risks.** The only risk listed is not a risk, so it should be cut. It ends "That matches what a loom start would do with the same binary", which concludes there is no hazard. Plan-Write needs no requirement from it. Cut the line and leave the section empty or with a "none" line. If the build note matters, move it to the support log.

**LOW — Scope, Out.** The first Out bullet repeats a rejected alternative. "Reordering `lym beam start` or the pawl recipe so the runs strand comes up before the loom start" is already the support log's first rejected alternative. The other two Out bullets mark real boundaries and should stay. This one re-argues a choice that was not made, which the compaction rules keep out of the record.

**LOW — D1 and Constraints.** The never-fail rule is stated twice. D1 says a failure "never fails `lym beam start` and never undoes the runs strand", and the first Constraint repeats "`ensureRunsStrand` never fails `lym beam start`…". Keep it in one place: D1 for the new add's failure, with the Constraint covering the existing behaviour, or drop the D1 half-sentence.

**LOW — D2 rationale, Constraints and Scope.** "No reed change" appears three times: as Scope Out "Any reed change", in D2's rationale "`Display.Window` already names a strand's window in reed, so no reed change is needed", and implied by the `ReplaceStrand` Constraint. Keep the Scope line and the reed facts in Constraints, and trim the D2 clause.

**LOW — Auto-mode assumptions, bullets 2 and 3.** These repeat D3:
- Bullet 2 ("overview line is built, though the entry says 'may'") repeats D3's rationale.
- Bullet 3 ("halted is not finished") repeats D3's "A halted run shows its line too".
- Keep the auto-pick fact once, either in D3 or as the assumption, not both.

**LOW — Goal.** The Goal carries cause narrative. Its second sentence retells the displacement mechanism (pawl loom start, `ensureRunsStrand`, `ReplaceStrand`), which D1's rationale already covers. The completeness-before-leanness test passes, since the plan writer needs the cause somewhere. One copy is enough: the Goal could keep only "the runs strand displaces the checklist" and leave the mechanics to D1.

Kept on purpose: the Notes for the plan writer are optional hints and fine. The `relevant.md` entries belong there.

I made no further tool calls.
