# Card 6 — statuspane-build-identity

**Create:**
- `plan:internal/statuspane#BuildIdentity` -> `type BuildIdentity struct`
- `plan:internal/statuspane#IdentityOf` -> `func IdentityOf(path string) (BuildIdentity, error)`
- `plan:internal/statuspane#CurrentBuild` -> `func CurrentBuild() (BuildIdentity, error)`
- `plan:internal/statuspane#Sidecar` -> `type Sidecar struct`
- `plan:internal/statuspane#ReadSidecar` -> `func ReadSidecar(path string) *Sidecar`
- `plan:internal/statuspane#WriteSidecar` -> `func WriteSidecar(path, guid string, build BuildIdentity) error`
- `plan:internal/statuspane#StrandCurrent` -> `func StrandCurrent(live bool, guid string, sidecar *Sidecar, build BuildIdentity) bool`
- `internal/statuspane/build_test.go`

**Delete:**
- `internal/loomcli#buildIdentity`
- `internal/loomcli#statusSidecar`
- `internal/loomcli#identityOf`
- `internal/loomcli#currentBuildIdentity`
- `internal/loomcli#readStatusSidecar`
- `internal/loomcli#writeStatusSidecar`
- `internal/loomcli/statusbuild_test.go`

**Edit:**
- `internal/loomcli#resolveStatusStrandAction` -> `func resolveStatusStrandAction(strands []reedengine.StrandStatus, sidecar *statuspane.Sidecar, current statuspane.BuildIdentity) (statusStrandAction, string)`
- `internal/loomcli#loomCLI.ensureStatusStrand`
- `internal/loomcli#TestResolveStatusStrandAction`
- `cmd/lyx/testdata/test-budget.yaml`

**Intent:**
Move the build identity and sidecar the loom bootstrap uses to keep or replace its status strand from `internal/loomcli/statusbuild.go` into `internal/statuspane/build.go`, so card 10's `runs` strand is kept or replaced by the same code (D1: "the way the loom bootstrap replaces a `loom-status` strand from an older build").

The moved code keeps its behaviour and its JSON keys, so a sidecar an older lyx wrote still reads: `BuildIdentity` (the symlink-resolved path, size and modification time), `IdentityOf`, `CurrentBuild` (of the running executable), `Sidecar` (`GUID`, `Build`), `ReadSidecar` (nil on a missing or unreadable file) and `WriteSidecar` (creating the parent directory, refusing an empty guid).
`StrandCurrent` is the one keep test both callers share: a live strand is current only when the sidecar names its guid and records `build`.
`statusbuild.go` keeps only `statusSidecarFile` and `statusSidecarPath`, loom's own path.

`resolveStatusStrandAction` takes the statuspane types and keeps the strand when `StrandCurrent` holds, with its three outcomes unchanged; `ensureStatusStrand` calls `statuspane.CurrentBuild`, `statuspane.ReadSidecar` and `statuspane.WriteSidecar`, and its doc comment calls it `start`'s status-strand work rather than the go arm's, the wording card 7's both-driver bootstrap needs, since card 7 does not touch the method again.

Tests: `build_test.go` carries `TestIdentityOf`, `TestSidecar_RoundTrip` and `TestReadSidecar_UnreadableIsNil` over from the deleted `statusbuild_test.go`, retargeted to the exported names; `TestResolveStatusStrandAction` builds its sidecars and identities with the statuspane types and keeps every row.
`StrandCurrent` is reached through those rows, which hold a matching sidecar, a changed build and a foreign guid.
`test-budget.yaml`: `internal/statuspane` 2 → 5.

**ImpactSummary:** the loom bootstrap's sidecar code moves package with its behaviour and on-disk shape unchanged; three tests move with it.
