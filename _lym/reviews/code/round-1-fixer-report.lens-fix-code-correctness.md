```yaml
lens: fix-code-correctness
outcome: reported
```

Task: check the correctness of the fixer's code changes (`fixScopeRules` string, removed test assertion, comment edits).

No findings.

- **`fixScopeRules`, source branch:** the string pieces join with correct spacing. Each piece ends in a space or `\n\n`. The rendered text reads: "...location and fix keys name. ... joins that whole group, so every finding lands in exactly one commit. Fix a group's findings together ... names every id in it: `...`. A finding that shares no changed file ... own commit. A finding whose change reaches a file an earlier commit already changed is committed on its own, never folded into that commit. Never push."
- **Callers:** the signature is unchanged. Both callers, `prompt.go:207` and `fix_test.go:206`, render the same function, so they cannot drift.
- **`prompt_test.go`:** the removed `requireNotContains(..., "commit each fix")` could never fail, because no scope renders that phrase. The "no git" assertions still pin the overlay/source split. The fix added no test, and none was needed since the finding was not a coverage gap.
- **Gate:** `lym gate test ./internal/burlerengine` passed, 634 passed and 0 failed.
- **Comments:** the `doc.go`, `verdict.go` and `prompt.go` edits are comment-only and cannot affect behaviour.

Further tool calls: none. My context already held every changed file.
