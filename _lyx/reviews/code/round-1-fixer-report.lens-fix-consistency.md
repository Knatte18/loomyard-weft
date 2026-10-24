```yaml
lens: fix-consistency
outcome: reported
```

Task: check the F1/F2 doc edits for stale twins elsewhere.

**Findings**

1. LOW — `docs/overview.md:518` (the `--amends-plan` bullet, two lines above the F2 edit) still reads "appends the entry's title to the plan amendment record, committed with the entry".
   - This is the same singular wording F1 corrected in `internal/loomcli/doc.go`.
   - `internal/loomcli/decision.go:538` already says "each entry's title ... committed with the entries".
   - The F2 edit made the notice bullet plural, so this sibling bullet is now the odd one out.

No other twins found. `decision.go:22` and `decision.go:538` already use the plural.

Tool calls made (1):
- `grep` for "entry's title/heading" across `.go` and `.md` files: my context did not hold the other sites that use this wording.
