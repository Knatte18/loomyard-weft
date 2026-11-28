# Card 3 — required-seam-installer

**Create:**
- `plan:internal/fabricengine#RunRecordPaths` -> `type RunRecordPaths struct`
- `plan:internal/fabricengine#SetRunRecordPaths` -> `func SetRunRecordPaths(paths *RunRecordPaths) (restore func())`
- `plan:internal/fabricengine#RunLocks` -> `type RunLocks struct`
- `plan:internal/fabricengine#SetRunLocks` -> `func SetRunLocks(locks *RunLocks) (restore func())`
- `plan:internal/fabricengine#StatusCodec` -> `type StatusCodec struct`
- `plan:internal/fabricengine#SetStatusCodec` -> `func SetStatusCodec(codec *StatusCodec) (restore func())`
- `plan:internal/fabricengine#suppliedSeams` -> `func suppliedSeams() map[fabriccore.RequiredSeam]bool`
- `plan:internal/fabriccli#InstallRequiredSeams` -> `func InstallRequiredSeams()`
- `plan:internal/hubforge#requiredSeamsOnce` -> `var requiredSeamsOnce sync.Once`
- `plan:internal/hubforge#InstallRequiredSeams` -> `func InstallRequiredSeams()`

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
- `plan:internal/shedengine#StatusLastMachine`
- `plan:internal/fabriccore#RequiredSeam`

**Intent:**
D13's required seams reach every process that runs fabric on a hub through one installer.

In `internal/fabricengine`, a new file holds the process-installed required seams, each an `atomic.Pointer` like `installedWorktreeSeam` (`worktreeseam.go`), in fabric's own vocabulary:

- `RunRecordPaths` carries `RunsRoot func(prime *lymcwd.Location) string`, `Reserved func(runID string) bool`, `StatusFile func(prime *lymcwd.Location, runID string) string` and `TreeStatusRel func(runID string) (string, error)`, the slash path of a run's status record in a records tree.
- `RunLocks` carries `PrimeRunLock func(l *lymcwd.Location) string`, the hub's prime run lock path.
- `StatusCodec` carries `LastMachine func(data []byte) (string, error)`.
- Each setter stores its value and returns a restore that puts back the value before it.
  A nil argument unsets the seam, so a test unsets one through the setter its production install calls.
- `suppliedSeams` reports, per `fabriccore.RequiredSeam`, whether its seam is installed.
  `snapshotReader.read` gains the `FieldSeams` case, which fills `snapshot.Seams` from it in any phase and never fails.

No verb reads a seam in this card, so behaviour is unchanged; cards 4, 9 and 10 move the shed calls onto them.

`fabriccli.InstallRequiredSeams` installs shed's adapter for all three:
the paths from `shedrun.RunsRoot`, `shedrun.IsReserved`, `shedrun.StatusFile` and the status half of `shedrun.TreeRunFileRels`;
the prime run lock from `shedrun.PrimeRunLock`;
the codec from `shedengine.StatusLastMachine`.
`internal/fabriccli` already imports both shed packages (`roleguard.go`, `anchordeps.go`), so its debt grows only by these adapters, as D13 allows.
`cmd/lym/main.go`'s root pre-run calls it inside the `!testing.Testing()` block beside the other fabric installs, and `TestPersistentPreRunE_InstallsSeams` adds `fabriccli.InstallRequiredSeams` to the calls it requires.
`internal/hubforge` calls it once per test binary through `requiredSeamsOnce.Do`, at the top of `cloneHub` and of `copyTemplate`, the two paths every hub-building export reaches, and never restores it.
`hubforge.InstallRequiredSeams` runs that same once, for a fabric test whose hub is built through `fabricengine.CloneHub` or `fabriccli.CloneAndWire` directly.
`grep -ln 'ReconcileOrigin\|OriginRunStatus\|OriginCodeTip' internal/fabricengine/*_test.go` lists the test files that run a seam-reading verb;
each of them whose hub no `internal/hubforge` builder makes calls `hubforge.InstallRequiredSeams` first, so no test meets the refusal unintended (D5).
`internal/hubforge` already imports `internal/fabriccli`, so its import set is unchanged.

Tests:
- `TestSnapshotReader_ReadsAHub` gains a `FieldSeams` row: on a hubforge hub every required seam reads supplied, which shows `internal/hubforge`'s one-time install reached the test binary.
- `TestPersistentPreRunE_InstallsSeams` holds the lym binary's call.
- The installer's adapter test against the real shed module is card 4's `TestInstallRequiredSeams`, once a verb reads the seams.

**ImpactSummary:** One new fabricengine seam file and one fabriccli installer; main.go and hubforge each gain one call, the pre-run guard one required name; no verb's behaviour changes in this card.
