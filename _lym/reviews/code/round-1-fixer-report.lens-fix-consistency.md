```yaml
lens: fix-consistency
outcome: reported
```

No findings.

Task: check the fix set for sibling or twin text left stale (fix-consistency).

I found no stale twin. Checked:
- **`removeIgnoredTaskPath` cross-references:** `doc.go:1331-1333` and `pattern/PATTERN-fabric-boundary.md:106-109` both name `removeIgnoredTaskPath` for the delete. Both still agree with the split into `ignoredTaskPathDeletable` (judge) and `removeIgnoredTaskPath` (delete). The PATTERN text is not wrong, so I raise no finding on it. `doc.go:1333` mentions the judging function, and the PATTERN entry does not.
- **`ErrStrayPathKept` sources:** the sentinel comment in `remove.go:52-55` and the `doc.go:793` bullet both now name `Add`, so they agree with the `Add` doc comment and `doc.go:1002-1005`.
- **`ErrTaskPathTaken` consumers:** `battencli/wire_test.go:779-780` matches the guidance text "the directory ... stands where the task worktree goes and is not a worktree". That text comes from `battencli`, not from `ErrTaskPathTaken.Error()`, so the reworded error message breaks no pinned string. `battencli`'s text and the new type comment cover the same refusal set.
- **Message-string pins:** no test or doc elsewhere pins the old "worktree directory %q already exists" wording; my grep found only the line in `addadoptonly.go` that was reworded.

Tool calls:
1. grep for `removeIgnoredTaskPath`, `ignoredTaskPathDeletable` and ignore wording across the repo. My context lacked every other use of these names.
2. grep for `PATTERN-fabric-boundary` files and `ErrTaskPathTaken` use in `battencli`. My context lacked the consumers and the PATTERN file.
3. grep of the PATTERN file for ignore and leftover text. I needed the line numbers of the entry.
4. Read of `PATTERN-fabric-boundary.md:95-113`. My context did not hold the entry's text.
