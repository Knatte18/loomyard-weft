```yaml
lens: fix-code-correctness
outcome: reported
```

Task: fix-code-correctness check of the code the fix changed (`isDocumentField`, `qualifiedPrefixes`, the allowlist keying, the new detector test).

1. **MEDIUM**: `isDocumentField`, `internal/configreg/keynames_enforcement_test.go`. A `<key>: <problem>` message whose first word ends at a comma or `}` is still treated as a document field and passes the guard, so F6's premise is only partly resolved.
   - Take `"max_bounces: negative, got %d"`. `after` is `: negative, got %d`, and `value` is cut at the comma to ` negative`. That is one field, so the function returns true and nothing is reported.
   - `"max_bounces: invalid, want at least 0"` passes the same way.
   - The cut at `,` and `}` applies whether or not the name stands in a flow map (no `{` or `, ` before it is required).
   - The doc comment says "A colon followed by several words introduces prose", which is false for these inputs.
   - No row in `TestKeyNames_DetectorReportsOnlyBareNames` covers a comma after the first word.

2. **LOW**: same function. The common wrapped-error forms `"timeout_min: %w"`, `"max_bounces: %v"` and `"max_bounces: required"` have a one-word value, so they are skipped although the key is named bare in a message. This is no worse than before the fix, but it is part of the `<key>: <problem>` form F6 names, and the fixer-report should state the limit.

The rest holds, by reading the code (I ran nothing):
- **New rows:** `MessageAfterColon`, `TextJoinedAfterColon` and `QuotedUnsetArgument` would each fail with the old code.
- **`SourceLiterals`:** it exercises the import, struct-tag and `KeyName`-call skips.
- **Allowlist:** the new `<file>: <name>: <literal>` keys match the lookup string, and the parent's gate run reported no stale or unmatched entry.
- **Production and test sweeps:** the diff check the parent ran showed comment-only changes.

Tool calls made: none.
