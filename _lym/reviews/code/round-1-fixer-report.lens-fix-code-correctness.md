```yaml
lens: fix-code-correctness
outcome: reported
```

No findings. Both edits are prose only, with no code, signature or test added or changed.

- **Orch stencil line 188:** no Go test or doc pins the removed "such as `CLAUDE.md`" clause. The only hit for the surrounding wording is the edited line itself.
- **`PATTERN-parent-directive.md` line 13:** no test reads the PATTERN text. `TestStencils_NoEditRuleRestatement` (`contracts/stencils/parentdirective_test.go:143`) scans the registered stencil bodies, not the PATTERN file, so the reworded line cannot break it.
- **Stale phrase, outside this lens:** the `//testtiming:keep` comment at `parentdirective_test.go:141` and the test's error message at line 151 still say "the edit rule". They are test-side wording the review did not name, so they are not a finding here.

Tool calls made:
1. `grep -rnE` for the old phrases across `.go`, `.md` and `.yaml` outside `_lym/`: to see whether any test or doc pins the edited text.
2. `Read` of `parentdirective_test.go:130-169`: to confirm the PATTERN-pinning test asserts on stencil bodies, not on the PATTERN file. My context did not hold that file.
