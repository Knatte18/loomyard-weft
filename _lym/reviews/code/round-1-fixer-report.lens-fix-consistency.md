```yaml
lens: fix-consistency
outcome: reported
```

Task: check the fix-consistency lens over the change set.

Findings:

1. **LOW**: `internal/fabricengine/codelocation_test.go` has no file-level comment.
   - The directory holds many other test files, so the file-header rule in `code-quality` applies.
   - The file opens straight at `package fabricengine_test`. Siblings such as `weftwiring_test.go` open with a header comment.

Everything else I checked agrees:
- **Stencils:** the two stencils use the same refuse_raw condition and the same "report the skipped tests to your parent" fallback.
- **No-test line:** its text matches across `tierrun.go`, the `TestRunTierNoTestNotice` expected line and the `doc.go` description. A grep finds no copy of the old wording.
- **Raw-test rule:** the stencils' "never a raw test command" rule still stands, with the hand run as the one stated exception.

Tool calls:
1. `grep` for the old line text, the stencil phrases and `testtiming` markers, plus a listing of `fabricengine` test files. My context lacked a repo-wide search for twins of the changed text.
