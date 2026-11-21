```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the F1 proposal as carried out in `internal/hardenshed/reap_test.go`, namely the extended link-target subtest and the new unresolved-root subtest.

No findings.

- **Extended `EndsAnOrphanWhoseCwdIsUnderTheRootsLinkTarget`:** pid 13 names the written root only through `Argv`, with `Cwd` outside both roots. If `rootProcesses` replaced the written root with the resolved one, pid 13 would no longer match and `want` `[]int{12, 13}` would fail. Pid 14 lies outside both roots, so a match-everything change would also fail. The fix therefore catches the regression the review described. Under a link, the resolved root differs from the written root, so both match arms are exercised.
- **New `EndsAnOrphanUnderTheWrittenRootWhenTheRootDoesNotResolve`:** `SandboxRoot` of a fresh `t.TempDir` has no `.lym`, so `EvalSymlinks` fails at `reap.go:199` and `roots` stays the written root alone. Pid 12 must be ended and pid 13 must not. Dropping the fallback, or returning the `EvalSymlinks` error, would fail the test through the pid assertion or the `Reap() error` check. The proposal does not break any sibling row, because each builds its own `t.TempDir` and the subtests are parallel-safe.
- **Weakness not worth a finding:** the unresolved-root test depends on `Reap`'s other steps (hub list, `emptyRoot`) tolerating a missing root. The gate passed with it, so that holds today.

Further tool calls: none, since my context already held everything this lens needed.
