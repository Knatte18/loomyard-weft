```yaml
lens: fix-consistency
outcome: reported
```

Task: check the fix set for stale twins and cross-references (fix-consistency lens).

**Findings**

1. **LOW** — `internal/fabricengine/doc.go:101` and `internal/fabricengine/originreconcile.go:35` (the `PairMayBeAbsent` field doc)
   - Both still say the run scope refuses only when the task worktree "is not a registered worktree of the code repository".
   - F1 changed the behaviour: a registered but prunable worktree now counts as absent, and `originreconcile.go:622` was reworded to say so.
   - These two descriptions of the same rule were left unchanged and no longer match it. A reader of the package doc or the option doc would not learn that a stale registration also takes the absent path.

No other stale twins found:
- **Archive tag rename:** the repo-wide grep finds no remaining `archiveTagSHALen`, `archiveTagName` or `archiveTagTipLength`.
- **Hand-forming of the tag name:** the only code that still builds `ArchiveTagPrefix + slug + "/"` is prefix matching in `cleanupremotewarp.go:225`, `recordsreads.go:51` and `remoteleftover.go:219`. It does not build the tag name from a tip, so it is not a twin.
- **Prunable handling:** `originreconcile.go:663` already skipped prunable entries, so the new `!entry.Prunable` at `originreconcile.go:640` now agrees with it. Other callers in the grep already exclude prunable entries.

**Tool calls**
1. Bash grep for the old identifiers and for `ArchiveTagPrefix +` uses, because my context held only partial reads of the other files.
2. Same Bash call: grep for `ContainsFunc(.*samePath` and `Prunable`, to find twin registered-worktree checks.
3. Bash grep for `PairMayBeAbsent` and "not registered", to find docs that restate the rule F1 changed.
