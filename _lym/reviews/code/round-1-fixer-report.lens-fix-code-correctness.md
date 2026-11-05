```yaml
lens: fix-code-correctness
outcome: reported
```

Task: check the correctness of the code the fix changed (`checkSubpathBeforeReset` and its helpers, `fetchWeftBranchTip`, the `CloneHub` gating, and the added tests), by running concrete inputs against my context.

no findings

Each case below was run step by step against the changed code.

- **Anchor disagrees with the recorded one.** The weft has `.lym-anchor` `.`, and `--reset` is run with `--subpath elsewhere`.
  - The warp branch is `main`, and the remote has no `main-weft`.
  - The prime ref therefore falls back to `HEAD`, which carries the marker.
  - `adoptRecordedAnchor` returns the mismatch error, so the hub is untouched.
  - With the defect in place the hub is removed first and the sentinel disappears, so `SubpathDisagreesWithRecordedAnchor` fails.
- **Subpath absent from the warp.** The weft records no anchor, and the subpath is `backedn`.
  - Neither the prime ref nor the legacy read finds a marker, so the check falls to `warpHasDirectory`.
  - `ls-tree -d HEAD --name-only -- backedn` returns empty, which gives the create-path error before the reset.
  - With the defect in place the sentinel is gone, so `SubpathAbsentFromWarp` fails.
- **Subpath present in the warp.** `warpHasDirectory` finds `backend`, and the reset proceeds.
  - `ResetWithSubpathPresentInWarp` would also pass without the fix.
  - It guards against over-refusal, which is its purpose.
- **Binding mismatch.** Two fixture dirs hold same-basename warps, so the hub path collides.
  - `checkResetTarget` passes, then the probe returns `ErrWarpBindingMismatch` before any removal.
  - Reverting the ordering removes the hub and drops the sentinel, so `BindingMismatch` fails.
- **Old-order guard.** The weft is an ordinary repo and `ForceBootstrap` is false.
  - The first clone pushes no binding or anchor to the remote, so the probe reports `WeftLooksLikeWeft` false.
  - The guard refuses before removal, and without the fix the sentinel is gone, so `OldOrderGuard` fails.
- **Failure of each call in `checkSubpathBeforeReset`.**
  - `MkdirTemp`, a staging clone, `checkedOutBranch`, `ls-remote`, `fetch`, `ls-tree` and `show` each return a wrapped error.
  - Each of those returns before `resetHub` runs, and the deferred `RemoveAll` removes the staging dir on every path.
  - An unborn weft or warp is guarded by `ErrNoCommits` in `readAnchorMarkerAt` and `warpHasDirectory`.
- **`probeMainWeftHasPath` after its refactor.** Its behavior is unchanged.
  - A missing branch gives `false, nil`.
  - A fetch error is wrapped as before.
  - The only caller is `probeWeftBinding`, and its signature is unchanged.
- **`CloneHub` gating.** The new `os.Stat(hubPath)` guard runs only with `Reset` and an explicit subpath.
  - Callers without `Reset`, and the livestate tests that use fake URLs with `Reset` and no subpath, never reach it.
  - Gate tier 1 passed with the new tests running.

Tool calls made: none. My context already held every changed file and the gate result.
