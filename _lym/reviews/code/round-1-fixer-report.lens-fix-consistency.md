```yaml
lens: fix-consistency
outcome: reported
```

Task: check the fix-consistency lens over the F1 change (`internal/hardenshed/reap_test.go`).

**Findings**

1. LOW — the subtest name `EndsAnOrphanWhoseCwdIsUnderTheRootsLinkTarget` (`reap_test.go:115`) no longer covers everything the subtest asserts.
   - The fix added process 13. It names the written root by `Argv` with a `Cwd` outside both roots, and the test now expects pids `[12, 13]`.
   - That assertion is about the written root still matching. It is not about the link target, so the name now understates the behavior. A reader scanning test names would not learn that the written root under a link is pinned here.
   - This is a naming gap only. The review's `fix` key asked for extending this subtest, so the extension is as proposed, and the name is the only stale sibling.

2. No other findings.
   - The new subtest `EndsAnOrphanUnderTheWrittenRootWhenTheRootDoesNotResolve` matches the review's wording. Its root is a fresh `SandboxRoot(t.TempDir())` with no `.lym`, so `EvalSymlinks` fails and only the written root is compared.
   - The `reap.go` fallback at `:199` (`err == nil && resolved != deps.Root`) agrees with both rows. The doc comment on `rootProcesses` ("the root, or the root with its links resolved") still holds, and no production code changed.
   - The twin `Reap` row `EndsRecordedProcessesThroughEnd...` is untouched and still consistent.

No further tool calls were made. My context already held the changed test and `reap.go`.
