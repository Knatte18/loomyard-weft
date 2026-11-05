```yaml
lens: fix-consistency
outcome: reported
```

Task: check the fix-consistency lens over the change set, looking for siblings or twins left out of step.

- **LOW** — `internal/fabricengine/recordsprobe.go:25`, the `warpProbeDirPrefix` doc comment.
  - It still reads "the os.MkdirTemp prefix for the probe's throwaway clone directory".
  - `resetsubpathcheck.go:25` now also takes its staging directory from this prefix, so the sentence covers only one of two users.
  - The `noProbeResidueInParent` doc at `warpbinding_clone_integration_test.go:37` and the `WarpProbeDirPrefixForTest` doc in `export_test.go:91-92` describe the prefix as the probe's alone.
  - The new `noProbeResidueInParent(t, cloneParent)` calls in the two subpath subtests make the staging residue part of the same contract.
  - Fix: widen the `warpProbeDirPrefix` comment to cover the staging clones.

- **LOW** — `internal/fabricengine/recordsprobe.go:1-9`, the file header.
  - It still describes the file as the pre-hub weft probe only, though the file now also holds `fetchWeftBranchTip`, which `resetsubpathcheck.go` uses.
  - This is minor, because the helper is a probe-shaped read.

- **LOW** — `internal/fabricengine/warpbinding_clone_integration_test.go:1-8`, the file header.
  - It lists "--reset in both argument forms" and the ordering guarantee.
  - It does not mention the new tests that pin refusals under `--reset`: binding mismatch, old-order guard and the two subpath refusals.
  - A reader choosing a file by its header would miss them.

- **LOW** — `internal/fabricengine/clone.go:236-244` against its `CloneHub` doc, `resetsubpathcheck.go` and the stale-marker error.
  - The pre-check mirrors the adopt and create paths but not the stale `.fabric-anchor` hard error at `clone.go:370`.
  - A `--reset` with an explicit `--subpath` against an old-layout weft with no `.lym-anchor` still passes the pre-check and then fails after the hub is removed.
  - That error is not an argument check, so this is a gap in the claim "a refused argument leaves the existing hub untouched" only for that legacy-marker case.
  - The doc comment as written ("after an explicit opts.Subpath has been checked") stays literally true.

No contradiction in the clone.go comments: the F5 wording (`clone.go:~235`) matches the code. The `livestate_*` comment rewrites agree with `checkResetTarget` being called from `CloneHub`. The `--reset` help string in `internal/fabriccli/fabric.go:97` and the docs stay accurate.

Tool calls made:
1. `grep` for subpath/reset/prefix references across Go and markdown files, to find other twins and docs that mention the changed behavior; my context did not hold files outside the changed set.
