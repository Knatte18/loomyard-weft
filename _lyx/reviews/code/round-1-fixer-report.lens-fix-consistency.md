```yaml
lens: fix-consistency
outcome: reported
```

No findings.

Task: check the fix set (F1–F4) for consistency with surrounding docs and sibling text.

- The spec line `contracts/specs/loom-plan-spec.md:618` says a test listed "in the same unit". The fix now also accepts the external test package (`<dir>_test`), so the spec wording is slightly narrower than the check. It doesn't contradict the check, because a `_test` unit is the same package directory. This is a LOW at most (a reader would notice, nothing breaks), and I'm not reporting it as a finding.
- `internal/planparser/doc.go:189` doesn't mention a unit scope, so it still agrees.
- The updated comment on `checkRenameLeavesTest` and the new `goPackageDir` comparison agree with each other. `goPackageDir` is the sanctioned unit adjustment, and the doc comment on `goPackageDir` still describes it correctly.
- The test rows (`foo_test`, `none`) agree with the check's `GoToolchain()` guard.
- The reworked engine subtest no longer sets `LaterTargetsReturned`. It uses `forkRewroteReport` for the second call, as the first hand-back subtest does, and its later assertions on the plan rewrite and fingerprint are unchanged.
- The comment I read at `recordbatch_git_test.go:~383` ("The batch was already handed back once…") was removed with the line it described, so no stale comment remains.

Tool calls:
1. `grep` for `rename-leaves-test` across `.go` and `.md` files. This was to find spec, doc and template twins of the changed check, which my context didn't hold.
