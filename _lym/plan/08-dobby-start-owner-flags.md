# Card 8 — dobby-start-owner-flags

**Edit:**
- `internal/dobbyengine#BeamPair`
- `internal/dobbyengine#StartDeps`
- `internal/dobbyengine#OwnedWorktrees`
- `internal/dobbyengine#EnsureLive`
- `internal/dobbyengine#Deps`
- `internal/dobbyengine#daemon.beamOwnedElsewhere`
- `internal/dobbyengine#fakeStart`
- `internal/dobbyengine#TestOwnedWorktrees`
- `internal/dobbyengine#TestEnsureLive`
- `internal/dobbyengine#newLoopHarness`
- `internal/dobbyengine#TestRun_ForeignOwner`
- `internal/hubgeom#DobbyStart`
- `internal/hubgeom#localBeams`
- `internal/beamcli#hubDobbyDeps`
- `internal/dobbyengine/doc.go`

**Create:**
- `plan:internal/hubgeom#beamPair` -> `func beamPair(slug string, owners boardengine.Owners, here machinename.Identity, parentOf func(slug string) (string, error)) (dobbyengine.BeamPair, error)`
- `plan:internal/hubgeom#TestBeamPair` -> `func TestBeamPair(t *testing.T)`

**Uses:**
- `internal/boardengine#OwnedElsewhere`
- `plan:internal/boardengine#EnsureIdentity`
- `plan:internal/machinename#ReadIdentity`
- `internal/hubgeom/doc.go`

**Intent:** Move the dobby's own name comparisons, `OwnedWorktrees`, `EnsureLive` and the loop's beam-owner read, onto the registry (D3: "the comparisons that name the machine directly move too").

`dobbyengine` stops comparing names: the hub wiring judges each owner through `boardengine.OwnedElsewhere` and hands the engine the answer.
`BeamPair` gains `Elsewhere bool`; `localBeams` builds each pair through `beamPair`, which reads the effective owner and judges it through `OwnedElsewhere` over the local Bolt's registry and the identity.
`StartDeps.PrimeOwner` returns `(owner string, elsewhere bool, err error)`, judged the same way in `DobbyStart`.
`StartDeps.Machine` stays, read only for the name `ClaimPrime` writes; `DobbyStart` fills it with the name `boardengine.EnsureIdentity` returns, so a machine whose config directory resolves from no variable starts no dobby (D3), and `EnsureLive`'s existing name error carries the refusal naming the variable.
`OwnedWorktrees` lists the prime when no prime owner is recorded or it is not elsewhere, and every local beam whose `Elsewhere` is false.
`EnsureLive` refuses a named prime or beam that reads elsewhere, as today with the owner and `lym adopt` named; a prime claim that stood reads here, and one lost to another machine reads elsewhere.
`Deps.Owner` returns `(owner string, elsewhere bool, err error)` and `beamOwnedElsewhere` reads the flag; `hubDobbyDeps` fills it through the registry.
`Deps.Machine` stays until card 23 retires the owner pause that still reads it.

Tests, tier0:
`fakeStart` and `newLoopHarness` carry the flags.
`TestOwnedWorktrees` and `TestEnsureLive` gain rows where an owner name equal to this machine's name reads elsewhere and a different name reads here, so the engine follows the flag and never the name.
`TestRun_ForeignOwner` reads the flag.
`TestBeamPair`, tier0, holds the D3 rows that feed `OwnedWorktrees`: a beam owned under a former name of this machine's registry entry reads here; an unregistered machine whose name is a registered machine's current name reads that machine's beam elsewhere; a beam with no owner of its own reads its parent's.
`dobbyengine/doc.go` says the wiring judges ownership through the registry.

**ImpactSummary:** StartDeps, BeamPair and Deps.Owner change shape; their one production filler each (hubgeom.DobbyStart, localBeams, beamcli.hubDobbyDeps) and the dobbyengine fakes follow.
