# Card 3 — required-seam-installer

**Create:**
- `internal/fabricengine#RunRecordPaths`
- `internal/fabricengine#SetRunRecordPaths`
- `internal/fabricengine#RunLocks`
- `internal/fabricengine#SetRunLocks`
- `internal/fabricengine#StatusCodec`
- `internal/fabricengine#SetStatusCodec`
- `internal/fabricengine#suppliedSeams`
- `internal/fabriccli#InstallRequiredSeams`
- `internal/hubforge#requiredSeamsOnce`
- `internal/hubforge#InstallRequiredSeams`

**Edit:**
- `internal/fabricengine#snapshotReader.read`
- `internal/fabricengine_test#TestSnapshotReader_ReadsAHub`
- `cmd/lym#newRoot`
- `cmd/lym#TestPersistentPreRunE_InstallsSeams`
- `internal/hubforge#cloneHub`
- `internal/hubforge#copyTemplate`

**Extends:**
- `internal/fabricengine/doc.go`
- `internal/fabriccli/doc.go`
- `internal/hubforge/doc.go`

**Uses:**
- `internal/shedengine#StatusLastMachine`
- `internal/shedengine#StatusRecorded`
- `internal/fabriccore#RequiredSeam`

**Intent:**
D13's required seams reach every process that runs fabric on a hub through one installer.

In `internal/fabricengine`, a new file holds the process-installed required seams, each an `atomic.Pointer` like `installedWorktreeSeam` (`worktreeseam.go`), in fabric's own vocabulary:

- `RunRecordPaths` carries `RunsRoot func(prime *lymcwd.Location) string`, `Reserved func(runID string) bool`, `StatusFile func(prime *lymcwd.Location, runID string) string` and `TreeStatusRel func(runID string) (string, error)`, the slash path of a run's status record in a records tree.
- `RunLocks` carries `PrimeRunLock func(l *lymcwd.Location) string`, the hub's prime run lock path.
- `StatusCodec` carries `LastMachine func(data []byte) (string, error)` and `Recorded func(data []byte) (bool, error)`, whether the bytes are a run status record shed wrote.
- Each setter stores its value and returns a restore that puts back the value before it.
  A nil argument unsets the seam, so a test unsets one through the setter its production install calls.
- `suppliedSeams` reports, per `fabriccore.RequiredSeam`, whether its seam is installed.
  `snapshotReader.read` gains the `FieldSeams` case, which fills `snapshot.Seams` from it in any phase and never fails.

No verb reads a seam in this card, so behaviour is unchanged; cards 4, 9, 10 and 11 move the shed calls onto them.

`fabriccli.InstallRequiredSeams` installs shed's adapter for all three:
the paths from `shedrun.RunsRoot`, `shedrun.IsReserved`, `shedrun.StatusFile` and the status half of `shedrun.TreeRunFileRels`;
the prime run lock from `shedrun.PrimeRunLock`;
the codec from `shedengine.StatusLastMachine` and `shedengine.StatusRecorded`.
`internal/fabriccli` already imports both shed packages (`roleguard.go`, `anchordeps.go`), so its debt grows only by these adapters, as D13 allows.
`cmd/lym/main.go`'s root pre-run calls it inside the `!testing.Testing()` block beside the other fabric installs, and `TestPersistentPreRunE_InstallsSeams` adds `fabriccli.InstallRequiredSeams` to the calls it requires.
`internal/hubforge` calls it once per test binary through `requiredSeamsOnce.Do`, at the top of `cloneHub` and of `copyTemplate`, the two paths every hub-building export reaches, and never restores it.
`hubforge.InstallRequiredSeams` runs that same once, for a fabric test whose hub is built through `fabricengine.CloneHub` or `fabriccli.CloneAndWire` directly.
`grep -rlE 'ReconcileOrigin|OriginRunStatus|OriginCodeTip|OriginParentRunStatus|OriginBattenStatus|ReadUpstreamTakeover|PushPairAnchored|CoalescePushRequest|CoalescePushBothAt|DetachedPush' --include=*_test.go internal cmd`
lists the test files that run a seam-reading verb, directly or through a records push's retry, in every package;
each of them whose hub no `internal/hubforge` builder makes calls `hubforge.InstallRequiredSeams` first, so no test meets the refusal unintended (D5).
Every hit outside package `fabricengine` builds its hub through a `hubforge` builder in the same process, `internal/fabriccli/weftskippush_integration_test.go` included, whose `CloneAndWire` mention is a comment.
Package `fabricengine`'s tests cannot call `internal/hubforge`, which imports `internal/fabricengine`.
Of its hits, `pushdeferral_integration_test.go` reaches `coalescePushSides` with no hub, and card 7 has its two tests set the run-records paths seam themselves;
`commit_partial_integration_test.go` only stubs the detached spawn (`swapPushRecorder`),
and `coalesce_test.go`, `spawn_test.go`, `spawn_linux_test.go`, `teardownlock_test.go` and `export_test.go` reach no seam read,
since they drive `coalescePush`'s loop, the spawn gating, the paused `DetachedPush` exit, or a stubbed detached spawn that never pushes (`export_test.go`'s `swapPushRecorder`).
`internal/hubforge` already imports `internal/fabriccli`, so its import set is unchanged.

Tests:
- `TestSnapshotReader_ReadsAHub` gains a `FieldSeams` row: on a hubforge hub every required seam reads supplied, which shows `internal/hubforge`'s one-time install reached the test binary.
- `TestPersistentPreRunE_InstallsSeams` holds the lym binary's call.
- The installer's adapter test against the real shed module is card 4's `TestInstallRequiredSeams`, once a verb reads the seams, with the run-lock check card 11 adds.
  The codec's `Recorded` adapter is `shedengine.StatusRecorded` itself, whose verdict card 2's `TestStatusCodec` holds;
  card 12's `TestReconcileOrigin_EndedRuns` reaches it through the installed seam over run records shed's writer made.

**ImpactSummary:** One new fabricengine seam file and one fabriccli installer; main.go and hubforge each gain one call, the pre-run guard one required name; no verb's behaviour changes in this card.
