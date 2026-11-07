```yaml
lens: leanness
outcome: reported
```

Lens: leanness. I looked for content that should move out of the decision record or be cut.

- **LOW, `## Notes for the plan writer`, first bullet:** it repeats the Constraints section. The `PATTERN-leaf-packages` and `PATTERN-test-isolation` constraint bullets already say each background gains the worktree entries and the registered template. Cut the note bullet, or cut the constraint clauses and keep only the note.
- **LOW, `## Notes for the plan writer`, third bullet:** "that parser stays for the reftable fallback" repeats D1's bound ("`List` falls back to the porcelain path, which stays for that case alone"). Keep the file pointer and drop the clause.
- **MEDIUM, `## PATTERN changes`:** both bullets prescribe overview lines identical to today's text, because the change is to the background only. A `changes:` bullet with unchanged text makes Webster rewrite a line to itself, and it hides the real edit, which only the Constraints and Notes describe. Either keep the bullets and say in each one that the background changes, or drop them and leave the background edits as the D1 and D2 work they already are. The record now says it in three places: PATTERN changes, Constraints and Notes.
- **LOW, `## Auto-mode assumptions`:** each bullet restates a decision's text (D7's merge-pair rule, D7's battencli `beamHub`, D4, D3's go-git, D1's reftable fallback). The section is required, but each bullet could name its pick and point at the D-id without restating it, since the support log's question ledger already holds the reasoning.
- **LOW, `## Constraints`, last bullet:** "Every D-numbered task runs its tests through `lym gate test`" restates `PATTERN-test-speed` and the repository's gate rule, and no decision departs from it. Cut it.
- **LOW, `## Constraints`, `PATTERN-child-processes` bullet:** "D1 and D3 remove processes and start none" carries no constraint, and for D1 it is wrong, because the reftable fallback starts git. Cut it, or narrow it to D3.
- **LOW, D3 bound:** "The repository has one `hubforge.SharedHub` site, a read-only one" is a count that will go stale, and "read-only" was not checked. The bound reads fine without the sentence, or it can say "today's `SharedHub` sites only read".
- **LOW, rationale figures in D1, D3, D4 and D7:** "1,272 copies at 16.6 ms", "about 5,600 processes a package run", "14.9 percent" and "9 to 11 s" copy the report's measurements, which the Goal already cites. Each rationale could keep one headline figure and cite the report section. This is emphasis only; the figures are correct as quoted.
- **LOW, outside my lens, D6 third bullet:** "fold only where reading the test confirms the overlap the survey names" hands the overlap check to the implementer. It sits close to the rubric's deferred-research-question flag. Naming those rows, or dropping them from scope, would settle it.

No further tool calls; my context held everything this lens needed.
