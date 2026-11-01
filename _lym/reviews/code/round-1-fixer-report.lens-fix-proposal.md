```yaml
lens: fix-proposal
outcome: reported
```

no findings

Lens fix-proposal: I challenged the two proposals the fixer carried out as written, F1 and F2.

- **F1** (`PrefixLongerThanForm` row in `internal/shellguard/rows_test.go`): the row holds up.
  - Removing the `len(prefixWords) <= len(words)` guard at `internal/shellguard/rows.go:230` makes the row panic. `strings.Fields` allocates `make([]string, n)`, so a three-word form has cap 3, and `words[:4]` goes out of range.
  - The expected text is the same as in `NoRefuseList`. That is correct, because a four-word prefix does not refuse the three-word forms, so both checks stay offered.
  - The row is not a duplicate of an existing case: only this row reaches the guard's false side.
  - I found no breakage elsewhere in the test.
- **F2** (shuttle paragraph of `docs/overview.md`): the wording holds up.
  - `refuse_raw` is the real key name. It appears in `internal/gateslot/doc.go:58` and in `internal/loomshed/plancontext.go:26`.
  - The new clause is still followed by `, and a third PreToolUse(Bash) guardrail`, so the sentence parses.
  - It matches the code's behaviour: the compile checks are offered only while `refuse_raw` does not refuse them.

Tool calls:
1. `grep -rn "refuse_raw"` over the repo: to confirm `refuse_raw` is the real config key, which my context did not show.
