```yaml
lens: leanness
outcome: reported
```

Leanness lens on the decision record and the support log: content to move or cut.

**Findings**

1. **MEDIUM. `## Auto-mode assumptions` repeats content the record already states elsewhere.**
   Each bullet restates a settled choice:
   - "`board-scope` and `repo-invariants` are the new lens names" is D1 and D2.
   - "replaces the pair in `review-discussion` … dormant `discussion` fan" is D1's second paragraph.
   - "`code_fixer_fan` stays a guard" is in the Out list under Scope and in D5's "other five … stay guards".
   - "`light`'s loom block runs the code review solo too" is D6.
   - "removed outright, with no alias" is D1 and D3's "delete".

   The question ledger in the support log (Q1 to Q5) holds these picks as well.
   Fix: shorten each bullet to a pointer at its decision, such as "D1: merge also applied to `review-discussion` and `discussion`, which the entry does not name".
   Alternatively, keep only the picks the entry does not license and drop the rest.

2. **MEDIUM. The `## PATTERN changes` bullet repeats the `PATTERN-config-layers` overview line unchanged, and a Note has to explain why.**
   A `changes:` bullet whose new text equals the current text says nothing on its own.
   It also invites a Code-Review "Declared PATTERN changes landed" check against a `PATTERN.md` line the diff never touches.
   D7's last bullet already carries the real change, the background's Guard bullet.
   Fix: say in one place that only the background file changes, either the bullet or D7, and drop the explanatory Note.
   The writer judges whether the spec needs a bullet for a change to the background only.

3. **LOW. The rationales in D1 to D4 and D6 restate figures from the audit, such as "9 of 46 and 14 of 38", "48 kept, 4 gating rounds, 9 dropped" and "65%".**
   The report is the source and the Goal already cites it.
   These figures are pinned snapshots that the plan writer does not act on.
   Fix: cite the report section, for example "audit §2 twill-discussion", and keep the one-clause reason without the counts.

4. **LOW. The docs bullet in Scope's In list duplicates D7's list of four docs.**
   Fix: keep one of them, for example "the docs D7 names".

5. **LOW. The second bullet of Notes for the plan writer, on fixture lens names in tests, is a call-site enumeration.**
   The rubric's "Do not flag" assigns that to Plan-Write's own lookups.
   It is harmless as a hint, but it is plan-level detail.
   Fix: cut it, or keep it as a one-line hint without the package list.

6. **LOW. The Constraints bullet "Every lens and fan this task changes lives in … no file under `contracts/stencils` names a changed lens" corrects the entry's wording.**
   It is a research fact the design rests on, so it stays in the record.
   Its supporting narrative already sits in the support log, Turn 2.
   No relocation is needed; I checked the completeness-before-leanness test and the bullet passes it.

Nothing else should move to the support log: Decisions carry no rejected alternatives, and Open risks carry facts for the parent.

Tool calls: none. My context held both files and the rubric.
