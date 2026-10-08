# Card 1 — Shared VCS build-identity reader

**Create:**
- `plan:internal/buildvcs#Identity` -> `type Identity struct`
- `plan:internal/buildvcs#Running` -> `func Running() Identity`
- `plan:internal/buildvcs#identityOf` -> `func identityOf(settings []debug.BuildSetting) Identity`
- `internal/buildvcs/doc.go`
- `internal/buildvcs/buildvcs_test.go`

**Edit:**
- `internal/shedverbs#binaryChanged` -> `func binaryChanged(running, recorded buildvcs.Identity) bool`
- `internal/shedverbs#inflightRecord`
- `internal/shedverbs#stepRecorder`
- `internal/shedverbs#stepRecorder.begin`
- `internal/shedverbs#newStepRecorder` -> `func newStepRecorder(dir, traceID string, build buildvcs.Identity) *stepRecorder`
- `internal/shedverbs#lastStepOf` -> `func lastStepOf(dir string, running buildvcs.Identity) *LastStep`
- `internal/shedverbs#statusCmd`
- `internal/shedverbs#stepCmd`
- `internal/shedverbs/buildidentity_test.go`
- `internal/shedverbs/status_test.go`
- `internal/shedverbs/steprecord_test.go`
- `internal/shedverbs/seam_enforcement_test.go`
- `docs/overview.md`
- `docs/shared-libs/README.md`

**Delete:**
- `internal/shedverbs#BuildIdentity`
- `internal/shedverbs#runningBuildIdentity`

**Intent:**
Decision 11: move the running binary's VCS reader out of `internal/shedverbs` into one shared leaf, `internal/buildvcs`, which `shedverbs` and card 6's `internal/hubreconcile` both import; `internal/buildinfo` stays import-free and is not touched.

`internal/buildvcs` (members in a new `buildvcs.go`, package doc in `doc.go`) imports only `runtime/debug`.
`Identity` is the VCS stamp of a lyx build, carrying exactly today's two fields and JSON tags: `Revision string` (`json:"vcs_revision"`) and `Modified bool` (`json:"vcs_modified"`); its doc says an empty `Revision` means the binary carries no VCS stamp.
`Running` reads `debug.ReadBuildInfo` and returns the zero value when build info is unavailable, otherwise `identityOf(info.Settings)`.
`identityOf` maps the `vcs.revision` and `vcs.modified` settings (`"true"` only) onto an `Identity`, ignoring every other key; it is split out so the mapping is testable without a stamped binary.
`doc.go` states the package's purpose (the VCS identity of the running lyx binary, read in-process) and that it is a stdlib-only leaf beside, not inside, `internal/buildinfo`, which stays import-free.

`internal/shedverbs` switches to `buildvcs.Identity`: `inflightRecord` embeds `buildvcs.Identity` in place of `BuildIdentity`, so the in-flight record's JSON keys stay `vcs_revision`/`vcs_modified` and a record written before this card still decodes; `stepRecorder.begin`'s record literal and `lastStepOf` use the embedded field under its new name.
`stepRecorder.build`, `newStepRecorder`, `lastStepOf` and `binaryChanged` take `buildvcs.Identity`; `statusCmd` and `stepCmd` call `buildvcs.Running()` where they called `runningBuildIdentity()`.
`BuildIdentity` and `runningBuildIdentity` are deleted; `buildidentity.go` keeps `binaryChanged` alone, and its file header comment is reworded to say so.
`LastStep`'s `vcs_revision`/`vcs_modified`/`binary_changed` fields are unchanged.

Tests: `buildvcs_test.go` tables `identityOf` over a settings list with both keys, `vcs.modified` `"false"`, no VCS keys at all (the zero value) and unrelated keys only.
`buildidentity_test.go`, `status_test.go` and `steprecord_test.go` replace their `BuildIdentity{...}` literals with `buildvcs.Identity{...}`; `TestLastStepOf_BuildIdentity` and `TestStatusCmd_LastStepCarriesBuildIdentity` keep asserting the same `vcs_revision`/`vcs_modified` values, which pins the acceptance criterion that step records carry the same fields after the move.
`seam_enforcement_test.go` adds `internal/buildvcs` to `shedverbsAllowedImports`.
`docs/overview.md`'s module tree gains an `internal/buildvcs/` line beside `internal/buildinfo/` ("the running binary's VCS identity, a stdlib-only leaf"), and `docs/shared-libs/README.md` gains a matching bullet beside `internal/buildinfo`'s.

**ImpactSummary:** Shedverbs-internal type swap: three re-signed helpers and two callers in the same package; the step-record and status JSON keys are unchanged.

**Commit:** 1: buildvcs: one shared reader of the running binary's VCS identity
