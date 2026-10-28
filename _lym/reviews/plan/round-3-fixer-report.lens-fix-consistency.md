```yaml
lens: fix-consistency
outcome: reported
```

Lens: fix-consistency. I checked whether the changed text in cards 7, 8 and 9 still agrees with the overview, the sibling cards and the decision record.

**LOW**: the decision record's D7 way-forward quote no longer matches card 7's string.
- **Changed text:** `_lym/plan/07-test-tier-check.md:43`, the way-forward string, which now inserts "or the word `untagged` when the file holds support only (TestMain, helpers) and carries no build line" after "such as `//go:build tier0`".
- **Summary:** D7 and card 7 now give a different way forward for the same finding. D7's quote, read as the finding's text, still leaves out the support-only form that the fix added.
- **Evidence:** `_lym/discussion/decision-record.md:122` reads "with the way forward: add an Intent line naming the file and its `//go:build` line, such as `//go:build tier0`." Before the fix, D7's quote was a leading prefix of card 7's string. After the fix, the two strings split right after `tier0`.
- **Not fixable here:** the decision record is outside the fixer's write surface, so this can only be recorded or deferred.

Cards 8 and 9 agree with card 7's rewritten Long-help sentence. Both point at "the sentence card 7 rewrote, the one starting 'Every mode but --require-approved also runs'". Both keep their check name out of the clause about what `rename-tool-missing` and `pattern-changes-mismatch` read, which matches card 7's move of that clause. Card 7 lands before 8 and 9, so the sentence exists when they edit it.

The set-of-tags sentence (line 41) agrees with the pass rule on line 42, with the spec row in the Docs paragraph and with the new two-lines-tier1 row. It also matches D7's "Together they name exactly one tier tag" and the parent decision.

The new helper row and the edited no-statement and `untagged` rows agree with the carry rule (line 44) and with the decision record's criterion at line 218.

Tool calls made:
1. I grepped the overview and cards 8 and 9 for way-forward, Long-help, `untagged` and "Every mode" text, and read the decision record's D7 section (lines 118–128). My context held the review's quote of D7, not D7's own way-forward sentence, and it had not shown the overview's text.
