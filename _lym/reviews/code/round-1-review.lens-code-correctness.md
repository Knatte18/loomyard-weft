```yaml
lens: code-correctness
outcome: reported
```

Lens: code-correctness. I checked each function and branch the diff changes by reasoning over the code in my context. I ran no tests.

**Per-branch list**

- `reconcileHubWide` snapshot of the overview (`walk.go:228`): input was a missing `HEDDLE.md`. `snapshotFile` stores `prior[path] = nil`, and `restoreConfig` removes the file on failure. Correct.
- The same snapshot with a stale file stores the old bytes, and `restoreConfig` rewrites them with `AtomicWriteBytes`. Correct.
- A read error on the overview aborts the write before any change, and `prior` already holds the config snapshot. Correct.
- Render and `written` (`walk.go:264`): a missing or stale file gives `changed == true`, so `HEDDLE.md` joins `written`. It is staged and committed even when no config file changed. `PullThenCommitWritten` only needs a non-empty `written`, so this works. Correct.
- Render with identical bytes: `RenderOverview` returns `(false, nil)`, so nothing is appended and nothing is committed. Correct.
- Pull skipped (board dirty or diverged): `PullThenCommitWritten` returns before calling `Write`, so nothing is rendered or snapshotted. Correct.
- Failure after the render (`afterHubWideWrite` error, commit failure): the error return goes through `failRestoringConfig(prior, …)` at `walk.go:277`. `prior` includes the overview, so the file is restored to its prior bytes or removed. Correct.
- `renderHeddleOverview` error path: `RenderOverview` returns the `ReadLayers` or read error before it writes anything, and the write is `fsx.AtomicWrite`. The file stays as it was, so the "leaves the file as it was" claim holds. The Warn call carries a `way_forward` key, matching the other calls in the package. Correct.
- Locking: the walk runs inside `PullThenCommitWritten`, which holds the Bolt write lock. `RenderOverview` takes no lock, and the replayer comment says the same. The heddle verbs hold the lock around it from outside. There is no re-entrancy or ordering problem. Correct.
- `snapshotFile` refactor of `snapshotConfig`: same ENOENT-to-nil and error-wrapping behaviour as the inlined loop, so no behaviour change. Correct.

**Findings**

1. LOW, class `design`, `walk.go:264`. `changed` compares the file with the bytes on disk, not with the committed bytes. When the board has no upstream, the pull step does nothing, so a dirty board is not skipped. If an earlier `lym heddle set` left `HEDDLE.md` rendered but uncommitted, the walk sees identical bytes and returns false. The file is then not in `written` and not committed by this walk, and the stamp is still written. The board sync of that earlier set commits it later, so the effect is small and self-healing. Evidence: `overview.go:41-44`, `bolt.go:247-261`, `walk.go:264-266`.

2. LOW, class `design`, `walk.go:264-266` and `walk.go:295-302`. A failed render is swallowed and the stamp is still written. A hub with an unloadable `heddle.json` then gets no retry until the next build or the next heddle write. The summary documents this as a deliberate trade-off, so I count it as a tolerated cost, not a defect.

No BLOCKING or MEDIUM correctness findings.

**Tool calls**
1. `grep` of the `git` helper and `boltStatusBefore` in `internal/heddlecli/cli_integration_test.go`, to check whether the helper trims porcelain output, because the new `wantBoltStatus` string ends in `"\n"`. It shows the helper returns `gitexec.Run` output unchanged. Whether `gitexec.Run` trims is not shown, so I could not settle it, and I raise no finding on it.
