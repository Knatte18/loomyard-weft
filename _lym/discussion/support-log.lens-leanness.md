```yaml
lens: leanness
outcome: reported
```

Task: review the decision record and support log for content to cut or move (leanness lens).

- **MEDIUM — Scope / Out.** The decision record carries a rejected alternative. The bullet "Records on ordinary branches of origin (support log, Rejected alternatives)" is a not-chosen design, not a scope boundary. The rubric puts rejected alternatives in `support-log.md`, which already has this one. Fix: cut the bullet. D1's choice of `refs/lym/` already fixes the scope.
- **LOW — Open risks, third bullet.** The bullet "No live push to GitHub's `refs/lym/` was made…; D3's refusal covers a host that rejects it" repeats two places: the last sentence of D1's rationale, and D3's refused-push refusal. It is a settled consequence, not an open risk. Fix: cut it, or keep it in the support log only (Q8 already says this).
- **LOW — Auto-mode assumptions / Open risks.** The assumption "The task's intent is read from its title, brief and operator scope note…" says the same as Open risks bullet 2. Keep it in one place: the Open risks bullet, since it asks the parent to confirm. The assumption line can then shrink to a pointer.
- **LOW — D6, last bullet.** "Any other general helper … stays only when a production path that remains after D5 calls it" restates `PATTERN-test-only-code`. That rule is already cited in D5's rationale and under Constraints. Fix: cut the bullet, or cut the Constraints line, not both.
- **LOW — D1 rationale.** The evidence names "(support log, turn 2)" as its pin, but the plan writer never reads the support log. The inline evidence (git version, refspecs, observed results) must stay: the completeness test and the unpinned-tool-fact rule keep it. So the support-log pointer adds nothing and can go. The same applies to "(support log, turn 2)" in D4.
- **LOW — Goal, last sentence.** "The redesign removes what f06acf39 built only to keep records on the code branch, and keeps its general pieces" previews D5 and D6. It is acceptable as a goal statement, but it could be cut without loss.
- **LOW (outside the lens) — Acceptance, D10 bullet.** It uses the `Review:` prefix for an absence of code, not for prose wording. The spec reserves `Review:` for text a stencil, spec or doc contains. Consider phrasing it as behaviour a test can check, or accept it as a review check knowingly.

Nothing I found must leave the record because it carries plan-needed requirements. The D1 evidence and the D3 refusals pass the completeness test and stay.

Further tool calls: none.
