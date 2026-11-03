```yaml
lens: fix-affected-set
outcome: reported
```

No findings in the fix as it stands against the review's text, but two of the review's three production `DiskSizes` call sites are still unpinned.

F1's set is the three production `DiskSizes(...WorktreeRoot)` reads. This is what each one has:

- **`internal/webstercli/cli.go:292`** is pinned by `TestExecutionBatches_SizesReadFromWorktreeRoot`. The test calls `executionBatches` with a nil state, and the probe finds `backend/README.md`. Swapping that call's root to `PaneCwd` fails the test.
- **`internal/webstercli/rebaseline.go:110`** has no test through the CLI that reads a size. The fixture's identity batcher never calls `sizes.Lines`, and the new test does not reach this call. Swapping this root to `PaneCwd` still passes every test.
- **`internal/websterengine/runlevel.go:511`** has the same gap. `runlevel_test.go` runs at an unanchored root, so anchor and worktree root coincide, as the review already noted. Swapping this root also passes.

The review's `fix` text says a root swapped in any of the three sites "then fails the test". The one test the fixer added only fails for `cli.go`. The premise is a set of three sites, and the fix covers one. I rate this MEDIUM, because it is the same coverage gap F1 describes, left open for two of its three members.

**Changed text:** `TestExecutionBatches_SizesReadFromWorktreeRoot` in `internal/webstercli/verbs_test.go`.

**Not covered by this lens:** the other `DiskSizes` hits (`cardgate_test.go:132`, `runlevel_test.go:2476`) are test code, and `runlevel_test.go:2476` builds its own root, so it adds no production coverage.

Tool calls:
1. `grep -rn "DiskSizes("` over `internal`, minus `internal/batcher/`, because my context had no list of every call site, only the three the review named.
