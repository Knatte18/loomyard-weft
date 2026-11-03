```yaml
lens: leanness
outcome: reported
```

Task: review the decision record and support log through the leanness lens, looking for content to move or cut.

**MEDIUM — Auto-mode assumptions, fifth bullet: this is behaviour, not an assumption, and the sentence is garbled.**
- "Records later review segments carry over into the decision record reach the board copy…" does not parse; "Records" should probably read "Carry-over blocks".
- The plan writer needs it: D8 and D9 copy the whole record, so Plan-Review and Code-Review carry-over blocks reach the board copy.
- Fix: state it once as a sentence in D8, which both verbs share, and drop the bullet.

**LOW — Auto-mode assumptions, bullets 1 and 2 repeat D1 and D6.**
- "The record file is named `decision-&lt;slug&gt;.md`" is D1. "Storing goes through the approve seam rather than a new recipe row" is D6.
- The rejected row alternative is already in the support log's Rejected alternatives and question ledger (Q1, Q2).
- Fix: cut both. The completeness test passes, because D1 and D6 already carry the requirement.

**LOW — Constraints: three bullets only say nothing changes.**
- "`PATTERN-sparse-hub-base`, `PATTERN-config-layers`: untouched" and "`PATTERN-project-record`: unchanged" give the plan writer nothing to act on.
- The project-record point is already in Scope Out ("Beam project records… stays in the beam's worktree") and in D8 ("a beam's project record is never copied").
- Fix: cut both bullets. "No PATTERN entry is added, changed or removed" covers them.

**LOW — Constraints: the `boardengine.OpenHub` bullet is a plan-level wiring hint.**
- It is not a constraint. D6 already ties the seam to the run-status precedent.
- Fix: move it to Notes for the plan writer, next to the `ApprovePlan` wiring note, or cut it.

**LOW — D1 and Constraints both say the record file's names are named constants.**
- D1: "The file prefix and suffix are named constants in `boardengine`". Constraints: "`PATTERN-named-constants`: the record file's prefix and suffix are declared once in `boardengine`."
- Fix: keep one. The Constraints bullet is enough, and D1 keeps only where the constants sit, next to the design-file constants.

**LOW — D7's rationale and the Open risks bullet say the same thing.**
- Both say `shed-sub-run-block` builds `cartoon` in parallel and that whichever task lands second fails the guard.
- Fix: keep the mechanism in D7's rationale. Cut the Open risk down to the question for the parent: is the guard, rather than a hand merge, the agreed coordination?

**LOW — The Goal's last sentence repeats Scope Out.**
- "This is the first half of the core of note `discussed-task-split`; skipping Discussion for a discussed entry is a later task." Scope Out already excludes the skip.
- Fix: keep the note reference and drop the clause after the semicolon.

**LOW — Notes for the plan writer, third bullet: precise placement is close to plan detail.**
- It says D8's copy goes "after the existing post-commit check, before the notices", which is close to the file:line-level detail the plan writer works out.
- It does not conflict with the record. Keep it as an optional hint; this is emphasis only.

No other findings. The Rejected alternatives and Question ledger sections are lean and sit in the right file, and no decision carries deliberation that belongs in the support log.

Tool calls: none. My context already held both files and the rubric.
