```yaml
lens: fix-affected-set
outcome: reported
```

**Findings: none.** I reran each finding's set search, and the fix covers every member I found.

- **F1 (callers of `ResolveLymBinary` with an injectable lookup).**
  - The non-test callers are `buttonclick.go:156` and seven in `panebin.go`.
  - The review already says `panebin_test.go` and `bindings_test.go` hold test-binary rows for the `panebin.go` callers.
  - The new `ConfirmEntryHandedTheTestBinaryRaisesNothing` row covers the `buttonclick.go` caller.
  - `buttonclick.go:71` and `lock.go:77` set `executablePath` to `selfexec.LymBinary` in production, so the row reaches the guarded path.
- **F2 (default `selfexec.LymBinary` lookups behind a nil seam).**
  - `loomcli/looparm.go:75` and `reedcli/window.go:86` are the two nil-defaults, and both now have a nil-lookup row.
  - The other nil-style defaults are `fronttask/hub.go:111`, `orchcli/cli.go:252`, `reedengine/buttonclick.go:71` and `reedengine/lock.go:77`.
  - The first is the case the review explicitly declined to raise.
  - The other three are set at construction, not via a nil check.
  - `ideengine/spawn.go:37` and `boardengine/spawn.go:32` are package-variable seams, the same shape as the `fronttask` case. Neither is in the review's scope.
  - The remaining direct `selfexec.LymBinary()` calls are the spawn sites the review settled as correct. They have no nil seam, so no "nil lookup" row applies.
- **Only note:** `loomcli/cli.go:195` is a direct `LymBinary()` call inside `loomcli`, with no seam. It is the same kind as the spawn sites the review settled. I raise no finding.

Tool calls:
1. Ran a grep over non-test files for `ResolveLymBinary(` and `selfexec.LymBinary`. My context had no site inventory for F1 or F2.
2. Ran a grep over test files for `os.Executable`, `ErrTestBinary` and `could not resolve`, and a grep for `lymBinary` in the other test files. I used them to see which existing tests already pin the test-binary refusal.
