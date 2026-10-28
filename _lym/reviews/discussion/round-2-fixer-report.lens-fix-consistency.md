```yaml
lens: fix-consistency
outcome: reported
```

Task: check that the round-2 fix set in `decision-record.md` agrees with the sibling decisions, criteria and cross-references it touches.

Changed or depended-on items checked: D4 glyph bullet, D4 `pkg.Member.Sub` bullet, D4 ambiguous and rejected sentence, D4 Bound, D5 nested-marker sentence, D5 scan, D5 Bound and Rationale, D6 cover bullets, D6 way forward, D6 Bound, D8 classification, D8 Bound, D9 index sentence, Auto-mode assumptions, and the acceptance bullets for D4, D5, D6 and D8.

1. **LOW.** Changed text: the D5 Bound now reads "reflection reached through any non-test helper, in the same package or another, is not seen". Problem: the D5 acceptance bullet was edited this round but still says "directly or through a same-package helper". It should say "same-package `_test.go` helper". As written, the bullet reads as covering the same-package non-test helpers that the Bound now excludes. Evidence: D5 trigger "through a same-package `_test.go` helper"; D5 Bound as changed; D5 acceptance "A test reflecting over a struct with `NumField`, directly or through a same-package helper, without `//lym:seam` fails the scan".

2. **LOW.** Changed text: the new D8 Bound sentence "A non-`Test` helper created by member glyph alone, in a card naming no `_test.go` file target, counts as non-test code, so its card lists the `doc.go`." Problem: the sentence just before it still says, without qualification, "a test-only card … not checked". A card that only creates such a helper is test-only, so the two Bound sentences disagree. F5 was raised to remove that disagreement. The earlier sentence needs an "except" clause, or the new one should be framed as the exception. Evidence: the two D8 Bound sentences.

3. **LOW.** Changed text: the new D4 Bound sentence "A field or method after the first member of a `pkg.Member.Sub` token is not tested." Problem: this narrows the entry's "every Go symbol" further than before. Scope Out records only the unqualified-name narrowing ("this narrows the entry's 'every Go symbol' to the shapes D4 can tell…"). Auto-mode assumptions does not list this narrowing either. F11 applied the same twin rule to D7's relaxation. Evidence: Scope Out last bullet; Auto-mode assumptions bullets.

No further tool calls; my context held every text I checked.
