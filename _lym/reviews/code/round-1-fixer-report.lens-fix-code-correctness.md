```yaml
lens: fix-code-correctness
outcome: reported
```

Lens: fix-code-correctness. Scope: the two `TestReap` changes in `internal/hardenshed/reap_test.go`, commit d2151726.

I found nothing to report. I traced `namesRoot` and `rootProcesses` for each row by hand, and the gate run passed.

**`EndsAnOrphanWhoseCwdIsUnderTheRootsLinkTarget`**
- Here `roots = [root, resolved]`, because the `.lym` link makes `resolved != root`.
- pid 12 has `Cwd = resolved/hub-one`. `namesRoot(…, resolved)` matches, since the end of the match is followed by `/`.
- pid 13 has `Argv` containing `root/hub-one`. It matches on the written root, and its `Cwd` of `/elsewhere` does not.
- pid 14 matches neither root, so it is not reaped.
- The expected `[12, 13]` is right, as long as `Reap` ends processes in list order.
- **Defect check, resolved root dropped:** a change that never adds the resolved root loses pid 12, so the row fails.
- **Defect check, written root replaced by resolved:** the same change loses pid 13, so the row fails. This is the escape F1 describes.

**`EndsAnOrphanUnderTheWrittenRootWhenTheRootDoesNotResolve`**
- The root has no `.lym`, so `EvalSymlinks` errors and `roots = [root]`.
- pid 12 (`Cwd = root/hub-one`) matches and pid 13 does not. The expected `[12]` is right.
- **Defect check, error propagated:** returning the `EvalSymlinks` error would make `Reap` return an error, so the `t.Fatalf` fires.
- **Defect check, written root not kept on failure:** pid 12 would be lost, so the row fails.
- It does not pin the empty-hub-list or missing-root paths in `emptyRoot`. The gate passed, so `Reap` tolerates a missing root.

**Callers:** the diff touches only test code, so no signature or caller changes.

**Minor, not a finding:** the unresolved-root row also passes on the merge base. The review asked for a pin of the fallback, not a red-first test, so that is acceptable.

**Tool calls**
1. `grep -n -A25 "^func namesRoot" internal/hardenshed/reap.go`: my context did not hold the `namesRoot` body, which I needed to trace the matches.
