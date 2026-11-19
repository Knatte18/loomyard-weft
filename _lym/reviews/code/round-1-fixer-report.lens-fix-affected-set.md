No findings.

- F2 (every `ListIssueComments` caller): two non-test callers. `intake.go:209` (`list --full`) is fixed; `intake.go:323` is the import path, where the request already names the issue, and the existing import test pins the unwrapped text. Not a gap.
- F3 (table rows): `CommentListFailure` is the only row that sets `failThread`, so it is the only row needing `wantErr`.
- F1 (sites stating the split rule): only `contracts/orchrole/role-intake.md` states it; the upsert bullet stays consistent with the new condition.

Tool calls (3):
1. grep for `ListIssueComments` in non-test Go files: to enumerate F2's callers.
2. grep for the split-rule phrases across md, go and yaml files: to enumerate F1's sites.
3. Read of `intake.go` lines 316–331: to see the second caller's error handling.
