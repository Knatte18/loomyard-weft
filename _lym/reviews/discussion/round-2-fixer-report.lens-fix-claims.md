```yaml
lens: fix-claims
outcome: reported
```

Checking the claims the fix added or rewrote against the code at this worktree's HEAD (fix-claims lens).

**LOW: D5 overstates when the weft pull stops on dirty paths**
- Changed text: D5's rationale says "without the commit every prime sync stops on the weft pull". D5's bound says "a change anywhere else under `_lym` stays dirty for preflight and still stops the weft pull".
- Both claim the stop on every sync and on every change. The code stops in only two cases:
  - the pull failed, the weft is not ahead of its upstream, and a tracked path is dirty;
  - both sides moved and the replay returns `ErrReplayDirty`.
- So a sync whose origin weft did not move never stops. An untracked new map, such as one from a first build or an import, never counts as dirty for the stop.
- Evidence:
  - `internal/fabricengine/pull.go:397-415`: the dirty stop is reached only after `pullErr`, and only with `len(ahead) == 0` and dirty paths present.
  - `internal/fabricengine/pull.go:440-445`: the `ErrReplayDirty` case.
  - `internal/fabricengine/pull.go:474-488`: `weftDirtyPaths` reads `trackedDirtyPaths` only.
- Fix: say "a prime sync whose origin weft moved stops on the weft pull", and "a tracked change anywhere else under `_lym` … can still stop the weft pull".

**Claims checked that hold**
- The D9 bound "the teardown's pending-records commit" takes the maps. `commitPendingRecords` stages `PathspecNames`, which always holds `structuralCommittedDirs` (`_lym`) (`internal/fabricengine/remove.go:792-845`, `junctionnames.go:26,59-61`).
- D5's claim that a map is "replaced atomically on disk" matches `testcover.WriteMap`'s contract (`internal/testcover/coveragemap.go:207-213`).
- D11 and support-log: "built at `5e41ee2b`, which has no Go change over `79f0eb4a`".
  - `git diff --stat 79f0eb4a 5e41ee2b -- '*.go'` is empty.
  - The bolt map `internal/lock/map.json` records `5e41ee2b06892e7cb9e0ca3090ddae27921cdf8b`, whose parent is `79f0eb4a`.
- The D7 claim that the redundancy step reads age and drift from `BuiltAt` and `Files` matches `internal/gatecli/redundancy.go:315-318,346-364`.
- D7's "as the redundancy sweep does" matches the pilot report, which runs a named `lym gate testmap internal/adoptcli` rebuild (`docs/research/speed/coverage-map-pilot-2026-11-12/report.md:10,36`).
- The `Constraints` line saying "write and build locks … through `lock.TrackedFileLockPath`" matches `WriteMap`'s `lock.TrackedFileLockPath(path, ".lock")` and `testMapBuildLockPath`.
- D4's `fabricengine.ReconcileOrigin` exists (`internal/fabricengine/originreconcile.go:787`).

**Tool calls**
- Read `remove.go`'s `commitPendingRecords` and `pull.go`'s `lockWeftForReplay`, to check the D9 teardown-commit bound and the D5 lock order. My context lacked those spans.
- Read `PathspecNames` and `structuralCommittedDirs`, to check whether the pending-records pathspec covers `_lym`. My context lacked them.
- Ran the `pathspecNames` body, the Go diff from `79f0eb4a` to `5e41ee2b`, the bolt map's commit field and a search for the sweep, to check the D11 rationale and D7's sweep claim. My context lacked these facts.
- Searched the pilot report for a named rebuild, to check D7's "as the redundancy sweep does". My context did not hold the report.
- Read `pull.go`'s `weftDirtyPaths`, to check D5's claims about when the weft pull stops. My context lacked that function.
