```yaml
lens: fix-consistency
outcome: reported
```

Task: check the fix set for stale siblings or twins, through the consistency lens.

**Finding 1: LOW.** Changed text: `contracts/stencils/webster/webster-body-implementer.md:131`, the sentence "A failure the card did not cause is never yours to fix, and it is left to the plan-level `webster` gate."
- Summary: the line was left unqualified while its two neighbours, lines 132 and 175, now carry the `--main-packages` exception.
- Evidence: line 107 says the exception overrides "the rules below", and line 131 states the rule flatly. A reader of the self-fix section can still take line 131 as controlling.
- The fixer may have judged this sufficient because 107 names the exception and 132 repeats it in the next sentence.
- The review's F3 fix key asked for the exception only "at the sentence that says such a batch gate failure does not make the report FAILED and at its repeat in the status rules", so this is outside the key. It is a small twin left unchanged.

No other stale twin found.
- The `template_test.go:863` pin matches the new line 175 text exactly.
- The `grep` shows no other stencil or Go source repeating the "did not cause" rule.
- The line 107 wording "rules below, in the self-fix bound and in the report's `status`" matches lines 132 and 175.

Tool calls:
1. `grep -rn "did not cause"` over `.md` and `.go` files: to find other copies of the "did not cause" rule that the fix might have missed. My context held only the stencil and `template_test.go`.
