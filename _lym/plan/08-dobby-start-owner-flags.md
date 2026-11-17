# Card 8 — dobby-start-owner-flags

**Edit:**
- `internal/dobbyengine#BeamPair`
- `internal/dobbyengine#StartDeps`
- `internal/dobbyengine#OwnedWorktrees`
- `internal/dobbyengine#EnsureLive`
- `internal/dobbyengine#Deps`
- `internal/dobbyengine#daemon.beamOwnedElsewhere`
- `internal/dobbyengine#fakeStart`
- `internal/dobbyengine#fakeStart.baseDeps`
- `internal/dobbyengine#TestOwnedWorktrees`
- `internal/dobbyengine#TestEnsureLive`
- `internal/dobbyengine#newLoopHarness`
- `internal/dobbyengine#TestRun_ForeignOwner`
- `internal/dobbyengine#runAnAdoptHold`
- `internal/dobbyengine#TestRun_ALaterArchiveFailureLogsAgainAfterTheOwnerLeaves`
- `internal/dobbyengine#TestRun_AFailedBoltPullLeavesTheOwnerReadToTheLocalBolt`
- `internal/dobbyengine#TestRun_BoltPullBringsInANewOwner`
- `internal/dobbyengine#TestRun_LockOwnerAndWakeups`
- `internal/dobbyengine#TestRun_TickFailuresLogOncePerEpisode`
- `internal/hubgeom#DobbyStart`
- `internal/hubgeom#localBeams`
- `internal/beamcli#hubDobbyDeps`
- `internal/beamcli#TestHubDobbyDeps_ABeamWithNoOwnerOfItsOwnIsUnderARootAdopt`
- `internal/beamcli#statusResult`
- `internal/beamcli#dobbyStatus`
- `internal/beamcli#statusCmd`
- `internal/orchcli#orchCLI.startDobbyStep`
- `internal/dobbyengine/doc.go`

**Create:**
- `plan:internal/hubgeom#beamPair` -> `func beamPair(slug string, owners boardengine.Owners, here machinename.Identity, parentOf func(slug string) (string, error)) (dobbyengine.BeamPair, error)`
- `plan:internal/hubgeom#TestBeamPair` -> `func TestBeamPair(t *testing.T)`
- `plan:internal/beamcli#TestDobbyStatus_ShowsTheIdentityRefusal` -> `func TestDobbyStatus_ShowsTheIdentityRefusal(t *testing.T)`

**Uses:**
- `internal/boardengine#OwnedElsewhere`
- `plan:internal/boardengine#EnsureIdentity`
- `plan:internal/boardengine#IdentityRefusal`
- `internal/machinename#ReadIdentity`
- `internal/orchcli#NotifyPrime`
- `internal/hubgeom/doc.go`

**Intent:** Move the dobby's own name comparisons, `OwnedWorktrees`, `EnsureLive` and the loop's beam-owner read, onto the registry (D3: "the comparisons that name the machine directly move too").

`dobbyengine` stops comparing names: the hub wiring judges each owner through `boardengine.OwnedElsewhere` and hands the engine the answer.
`BeamPair` gains `Elsewhere bool`; `localBeams` builds each pair through `beamPair`, which reads the effective owner and judges it through `OwnedElsewhere` over the local Bolt's registry and the identity.
`StartDeps.PrimeOwner` returns `(owner string, elsewhere bool, err error)`, judged the same way in `DobbyStart`.
`StartDeps.Machine` stays, read only for the name `ClaimPrime` writes; `DobbyStart` fills it with the name `boardengine.EnsureIdentity` returns.
A machine whose config directory resolves from no variable starts no dobby (D3: a refused registration at a dobby start posts one notice and shows in `lym beam status`):
`EnsureLive` reports a failed `Machine` read as an `OutcomeRefused` entry for each named worktree, in place of its name error;
the entry's `Detail` is the refusal, which names the variable to set and the `lym beam stop` and `lym beam start` restart.
The bring-up's `startDobbyStep` (`internal/orchcli`) keeps its warning for that entry and posts the refusal once per bring-up to the prime's orch through `NotifyPrime`.
`lym beam status` carries `machine_refused`, the refusal `boardengine.IdentityRefusal` reads, so the refusal shows while no dobby runs; `statusCmd`'s help names the field.
`OwnedWorktrees` lists the prime when no prime owner is recorded or it is not elsewhere, and every local beam whose `Elsewhere` is false.
`EnsureLive` refuses a named prime or beam that reads elsewhere, as today with the owner and `lym adopt` named; a prime claim that stood reads here, and one lost to another machine reads elsewhere.
`Deps.Owner` returns `(owner string, elsewhere bool, err error)` and `beamOwnedElsewhere` reads the flag; `hubDobbyDeps` fills it through the registry.
`Deps.Machine` stays until card 23 retires the owner pause that still reads it.

Tests, tier0:
`fakeStart`, `fakeStart.baseDeps` and `newLoopHarness` carry the flags.
`runAnAdoptHold` and the loop tests that set `Deps.Owner` (`TestRun_ALaterArchiveFailureLogsAgainAfterTheOwnerLeaves`, `TestRun_AFailedBoltPullLeavesTheOwnerReadToTheLocalBolt`, `TestRun_BoltPullBringsInANewOwner`, `TestRun_LockOwnerAndWakeups`, `TestRun_TickFailuresLogOncePerEpisode`) switch their fakes to the `(owner, elsewhere, err)` shape.
`TestOwnedWorktrees` and `TestEnsureLive` gain rows where an owner name equal to this machine's name reads elsewhere and a different name reads here, so the engine follows the flag and never the name.
`TestEnsureLive` gains a row whose `Machine` seam returns the identity refusal: every named worktree gets an `OutcomeRefused` entry carrying the refusal as its detail, and no dobby is spawned.
`TestRun_ForeignOwner` reads the flag.
`TestHubDobbyDeps_ABeamWithNoOwnerOfItsOwnIsUnderARootAdopt`, tier1 in `internal/beamcli`, reads `Deps.Owner`'s three values and asserts the beam reads elsewhere.
`TestBeamPair`, tier0, holds the D3 rows that feed `OwnedWorktrees`: a beam owned under a former name of this machine's registry entry reads here; an unregistered machine whose name is a registered machine's current name reads that machine's beam elsewhere; a beam with no owner of its own reads its parent's.
`TestDobbyStatus_ShowsTheIdentityRefusal`, tier1 over a hub fixture with `XDG_CONFIG_HOME` and `HOME` set empty once the fixture is built, asserts `machine_refused` names the variable and the restart; it sets process environment, so it does not run in parallel.
`dobbyengine/doc.go` says the wiring judges ownership through the registry.

**ImpactSummary:** StartDeps, BeamPair and Deps.Owner change shape; their one production filler each (hubgeom.DobbyStart, localBeams, beamcli.hubDobbyDeps) and the dobbyengine fakes and tests that set them follow; an unresolved identity refuses each dobby start as an entry, which the bring-up posts and beam status shows.
