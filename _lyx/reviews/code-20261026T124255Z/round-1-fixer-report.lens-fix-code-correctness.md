```yaml
lens: fix-code-correctness
outcome: reported
```

Task: check the fix's changed code through the fix-code-correctness lens.

The fix changed no executable code, so there is no function or branch to run step by step:

- `internal/shellguard/rows.go:61`: comment text only. The sentence now reads "through a run of shell units, and one after a heredoc operator counts only on that operator's line". `commandPositionPattern` and the constants around it are untouched.
- `internal/editdirective/editdirective_test.go`: the assertion is back to the single `strings.Contains(got, "Edit or Write")` check, and the doc comment is back to the anchor-phrase wording. The test's other checks (`{{` absent, no leading comment) are unchanged. The test still fails if the directive loses "Edit or Write", so it still catches its defect. It also no longer pins stencil prose, which is what F3 asked for.
- `pattern/PATTERN-refusal-way-forward.md` and `docs/research/codex-feasibility.md`: prose only. No Go code parses either file.
- Gate result: `lyx gate test ./internal/editdirective ./internal/shellguard ./cmd/lyx` passed for all three packages, including the `norecoverydeny` test that reads the closed list.

No findings.

Further tool calls: none. My context already held every changed file and the gate output.
