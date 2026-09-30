# Card 1 — shedrun-runs-root-rel

**Create:**
- `plan:internal/shedrun#RunsRootRel` -> `func RunsRootRel() string`

**Edit:**
- `internal/shedrun/paths.go`
- `internal/shedrun/paths_test.go`
- `internal/shedrun/doc.go`

**Intent:**
Add `RunsRootRel`, an anchor-relative constructor for the shed run-records root: `filepath.Join(lyxdirs.LyxDirName, shedDirName)`, the directory every run's `RunDir` sits under.
It takes no `*lyxcwd.Location`, because the root carries no run-id and so needs none of `runSegment`'s alias resolution.
It exists so `fabricengine`'s `Add` can drop the whole run-records tree from a freshly forked pair without naming the `shed` segment itself, per the Shed Run-Directory Invariant (Decision 4).
Its doc comment states that purpose and that the returned path is anchor-relative, the same shape `SeedRel`/`StatusRel` return, suitable for `fabricengine.CommitWeftPaths`' `relPaths`.

Update the `paths.go` file header's list of what the file declares to include the run-records root, and add one sentence to the package doc in `doc.go` naming `RunsRootRel` and why `fabricengine` needs it: a freshly forked pair drops everything under it.

Add `TestRunsRootRel` to `paths_test.go`: it asserts `RunsRootRel()` equals `filepath.Join(lyxdirs.LyxDirName, "shed")`, and that `SeedRel(l, "x")` and `StatusRel(l, "x")` both lie strictly under it (prefix `RunsRootRel()` plus a separator), pinning that the root the drop removes is the one every run's records live under.

**ImpactSummary:** Additive: one new exported function plus doc text in `internal/shedrun`; no existing caller changes.
