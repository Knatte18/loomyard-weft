# Card 7 — record the build identity per step; derive `binary_changed`

**Edit:**
- `internal/shedverbs/steprecord.go`
- `internal/shedverbs/steprecord_test.go`
- `internal/shedverbs/step.go`
- `internal/shedverbs/status.go`
- `internal/shedverbs/status_test.go`

**Create:**
- `internal/shedverbs/buildidentity.go`
- `internal/shedverbs/buildidentity_test.go`

**Intent:**
Give the driver a Go-derived "a new `lyx` was deployed" signal (decision record D4).

`buildidentity.go` declares an exported `BuildIdentity` struct holding `Revision string` (JSON `vcs_revision`) and `Modified bool` (JSON `vcs_modified`).
An empty `Revision` means the binary carries no VCS stamp.
It also declares:
- `runningBuildIdentity() BuildIdentity`, reading `runtime/debug.ReadBuildInfo`'s `vcs.revision` and `vcs.modified` settings, and returning the zero value when build info is unavailable;
- a pure `binaryChanged(running, recorded BuildIdentity) bool`, true only when both revisions are non-empty and the pairs differ in either field.
The read lives here rather than in `internal/buildinfo`, which the Buildinfo Leaf Invariant keeps import-free and limited to `Channel`/`IsDev()`.
`runtime/debug` is stdlib, so `shedverbs`' own import allowlist is unchanged.

`steprecord.go`: `inflightRecord` gains the identity, serialised as `vcs_revision` and `vcs_modified`.
`newStepRecorder` takes the identity to record as a parameter, and `begin` writes it; `stepCmd` (`step.go`) passes `runningBuildIdentity()`.
A record written before the fields existed decodes as an unknown identity.
`LastStep` gains `vcs_revision`, `vcs_modified` and `binary_changed`.
`lastStepOf` takes the running identity as a parameter, copies the chosen record's recorded identity onto `LastStep`, and sets `binary_changed` from `binaryChanged`.
An in-flight record that cannot be decoded reports an unknown identity and `binary_changed: false`.
The file's doc comments describe the new fields.
`status.go`'s status verb passes `runningBuildIdentity()` to `lastStepOf`.
The step envelope itself is unchanged.

Identities are passed as parameters rather than through a package-level seam, so tests substitute nothing shared.

Tests:
- `buildidentity_test.go`: a table over `binaryChanged` covers equal pairs (false), differing revisions (true), an equal revision with a flipped modified flag (true), and an empty revision on either side or both (false, whatever the other fields hold).
- `steprecord_test.go`: a recorder built with a given identity writes it into the in-flight record, and `lastStepOf` reports it back with `binary_changed` false for the same running identity and true for a different known one.
  A hand-written in-flight record with no identity fields reports `binary_changed: false` against a known running identity.
- `status_test.go`: the status envelope's `last_step` carries the `vcs_revision`, `vcs_modified` and `binary_changed` keys, with `binary_changed` false (a test binary is unstamped).

**ImpactSummary:** Adds two fields to each step's in-flight record and three to `last_step` in `lyx shed status`; `newStepRecorder` and `lastStepOf` each gain one parameter with a single production caller, and existing records still decode.

**Commit:** 7: shedverbs: record each step's build identity; status derives binary_changed
