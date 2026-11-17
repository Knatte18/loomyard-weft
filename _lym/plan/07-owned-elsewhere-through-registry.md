# Card 7 — owned-elsewhere-through-registry

**Edit:**
- `internal/boardengine#OwnedElsewhere` -> `func OwnedElsewhere(owner string, registry Registry, here machinename.Identity) bool`
- `internal/boardengine#writeOwners`
- `internal/boardengine#TestOwnersEffective`
- `internal/pairteardown#runReader`
- `internal/pairteardown#runReader.read`
- `internal/pairteardown#hubRunReader`
- `internal/pairteardown#runFakes`
- `internal/pairteardown#runFakes.reader`
- `internal/pairteardown#TestReadRun`
- `internal/pairteardown#OwnerReader`
- `internal/fabriccli#readLocalReading`
- `internal/fabriccli#foreignOwner`
- `internal/beamcli#landDeps`
- `internal/beamcli#createDeps`
- `internal/beamcli#hubLandDeps`
- `internal/beamcli#hubCreateDeps`
- `internal/beamcli#beamLandReading`
- `internal/beamcli#judgeLandOwners`
- `internal/beamcli#judgeCreateEntry`
- `internal/beamcli#judgeParentOwner`
- `internal/beamcli#create`
- `internal/loomcli#localOwnerVerdict` -> `func localOwnerVerdict(l *lymcwd.Location, slug string, identity func() (machinename.Identity, error), parentOf func(slug string) (string, error)) (ownerVerdict, error)`
- `internal/loomcli#runOwnerVerdict` -> `func runOwnerVerdict(l *lymcwd.Location, slug string, identity func() (machinename.Identity, error)) (ownerVerdict, error)`
- `internal/loomcli#RunOwnerVerdict`
- `internal/loomcli#newLoomCLI`
- `internal/loomcli#pawlDeps`
- `internal/loomcli#pawlRecheck`
- `internal/loomcli#pawlRecheckLines` -> `func pawlRecheckLines(slug string, ready bool, blockers battencli.BeamLandBlockers, refusal error, registry boardengine.Registry, here machinename.Identity, beamOwner, parentOwner, parentSubject string) []string`
- `internal/loomcli#TestPawlRecheckLines`
- `internal/loomcli#parentOwner`
- `internal/adoptcli#adoptDeps`
- `internal/adoptcli#hubDepsAs`
- `internal/adoptcli#hereRequest`
- `internal/adoptcli#subjectOwnedElsewhere`
- `internal/adoptcli#pauseTargets`
- `internal/adoptcli#pauseSubject`
- `internal/adoptcli#resumeSlug`
- `internal/adoptcli#foreignParent`
- `internal/adoptcli#stepReconcile`
- `internal/adoptcli#runReports`
- `internal/adoptcli#TestRunReports`
- `internal/tendercli#ownerElsewhere`
- `internal/tendercli#pairOwners` -> `func pairOwners(owners boardengine.Owners, slugs []string, here machinename.Identity, parentOf func(slug string) (string, error)) ([]tenderengine.PairOwner, error)`
- `internal/tendercli#HubDeps`
- `internal/tendercli#TestPairOwners`
- `internal/orchcli#readBeamFacts`
- `internal/orchcli#archiveBeforeOwnerRefusal` -> `func archiveBeforeOwnerRefusal(refusal error, owner string, elsewhere bool, archive func() error) error`
- `internal/orchcli#beamUpRefusal` -> `func beamUpRefusal(slug, here, owner string, elsewhere bool, moveWayForward string, held, ownLanding bool, status string) error`
- `internal/orchcli#orchCLI.beamUp`
- `internal/orchcli#TestBeamUpRefusal`
- `internal/orchcli#beamOwnerElsewhere`
- `internal/battencli#ownerSeams`
- `internal/battencli#hubOwnerSeams`
- `internal/battencli#battenCLI.thisMachine`
- `internal/battencli#fakeOwners`
- `internal/battencli#fakeOwners.seams`
- `internal/battencli#readTaskPlacement` -> `func readTaskPlacement(prime *lymcwd.Location, parents *DobbyParents, owners boardengine.Owners, here machinename.Identity, slug string) (parent string, ownedElsewhere bool, err error)`
- `internal/battencli#DobbyTasks`
- `internal/battencli#battenCLI.otherMachineOwning`
- `internal/battencli#battenCLI.refuseStartOwnedElsewhere`
- `internal/battencli#battenCLI.refuseOwnedElsewhereOnLocalBoard`
- `internal/battencli#battenCLI.ownerExtras`
- `internal/battencli#cancelDeps`
- `internal/battencli#judgeOrigin`
- `internal/battencli#hubCancelDeps`
- `internal/battencli#cancelFake.deps`
- `internal/fronttask#HubSettleDeps`
- `internal/boardengine/doc.go`
- `internal/pairteardown/doc.go`

**Uses:**
- `plan:internal/boardengine#Registry.ReadsAsHere`
- `plan:internal/machinename#ReadIdentity`
- `internal/fabriccli/doc.go`
- `internal/adoptcli/doc.go`
- `internal/fronttask/doc.go`

**Intent:** Route every comparison of a recorded machine name with this machine through the registry (D3).

`OwnedElsewhere(owner, registry, here)` reads false for an empty owner and otherwise `!registry.ReadsAsHere(owner, here)`.
So an owner recorded under a former name of a registered machine reads as owned here on that machine and as that machine on every other, and an unregistered machine whose name is a registered machine's current name reads that machine's owners as owned elsewhere.
`writeOwners` passes the loaded store's registry and `req.Identity`, or `machinename.Identity{Name: req.Machine}` when the request carries none.

Each caller passes the registry of the `Owners` it already read (`owners.Registry`) and this machine's `machinename.Identity`, read through `machinename.ReadIdentity` where it read `machinename.Name` for the comparison today.
How the identity reaches each comparison:
- A function that takes `here` as a name and no deps value is re-signed (arrows above), with its callers: `pairOwners` (`HubDeps`, `TestPairOwners`), `readTaskPlacement` (`DobbyTasks`), `pawlRecheckLines` (`pawlRecheck`, `TestPawlRecheckLines`), `localOwnerVerdict` and `runOwnerVerdict` (`newLoomCLI` fills the identity seam with `machinename.ReadIdentity`), `archiveBeforeOwnerRefusal` and `beamUpRefusal`, which take the `elsewhere` flag `orchCLI.beamUp` judges through the registry from `readBeamFacts` (`TestBeamUpRefusal` passes the flag; `beamUpRefusal` keeps `here` for its wording).
- A function that already takes a deps value keeps its signature and reads the identity from a new seam on that deps:
  `adoptDeps.Identity`, which `hubDepsAs` fills, read by `hereRequest`, `subjectOwnedElsewhere`, `pauseTargets`, `pauseSubject`, `resumeSlug`, `foreignParent`, `stepReconcile` and `runReports` (whose `reportInput.Machine` stays the name the reports print);
  `landDeps.Identity` and `createDeps.Identity`, which `hubLandDeps` and `hubCreateDeps` fill, read by `judgeLandOwners`, `judgeCreateEntry`, `judgeParentOwner` and `create` (`create`'s `entry.Owner != here` moves onto the registry too);
  batten's `ownerSeams.Identity`, which `hubOwnerSeams` fills and `thisMachine`'s neighbours read in `otherMachineOwning`, `refuseStartOwnedElsewhere`, `refuseOwnedElsewhereOnLocalBoard` and `ownerExtras`, while `ownerSeams.Machine` stays the name owner writes take.
  A nil `Identity` seam, as a hand-built deps or receiver has, reads `machinename.Identity{Name: <the name the deps' Machine seam or here gives>}`, so the comparison falls back to the name alone and every existing hand-built test keeps its meaning.
  `fakeOwners` gains an `identity` field that `seams` hands out, its zero value reading as its `machine`.
  `TestRunReports` hands an identity in through its deps.
- orchcli's `beamFacts` carries the elsewhere flag beside `here`, which `readBeamFacts` fills; `beamOwnerElsewhere` judges through the registry.
- batten cancel's `cancelDeps.Owner` returns `(owner, subject string, elsewhere bool, err error)`, judged through the registry in `hubCancelDeps`, so `judgeOrigin` no longer compares names, and `cancelFake.deps` returns the new shape.
- `fabriccli`'s `foreignOwner` and `readLocalReading`, and `fronttask`'s `HubSettleDeps`, read the identity directly.

`pairteardown`'s `runReader.owner` seam is replaced by `owners func() (boardengine.Owners, error)`, `identity func() (machinename.Identity, error)` and `parentOf func(slug string) (string, error)`;
`read` computes the effective owner, the adopt subject and `OwnedElsewhere` from them, so `TestReadRun` reaches the registry rule with fakes.
`runFakes` gains the `owners`, `identity` and `parentOf` fields in place of `owner`, `adoptSubject` and `elsewhere`, and `runFakes.reader` hands them to the reader.
`hubRunReader` wires them over the local Bolt, `machinename.ReadIdentity` and `hubgeom.ParentOf`; `OwnerReader` compares through the registry the same way.

Tests:
`TestOwnersEffective`'s `OwnedElsewhere` rows take a registry and an identity; the empty-owner row stays, and the registry cases stay with card 3's `TestReadsAsHere`.
`TestReadRun`, tier0, gains the D3 rows: an owner recorded under a former name of this machine's entry reads owned here; the same owner read by another identity reads owned elsewhere naming that owner; an unregistered machine whose name is a registered machine's current name reads that machine's run as owned elsewhere.
`TestPairOwners`, `TestPawlRecheckLines` and `TestBeamUpRefusal` hand in an identity or the flag.
`boardengine/doc.go` and `pairteardown/doc.go` describe the registry comparison and the new reader seams.

**ImpactSummary:** One re-signed function with about forty call sites across eleven packages; seven helpers re-sign with their callers and tests, and four deps types gain an identity seam whose nil value keeps the name comparison.
