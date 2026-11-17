# Card 5 — The orch watcher and the dobby read this machine's hold

**Create:**
- `plan:internal/orchcli#UnderOwnHold` -> `func UnderOwnHold(prime *lymcwd.Location) func(slug string) bool`
- `plan:internal/orchcli#TestUnderOwnHold` -> `func TestUnderOwnHold(t *testing.T)`

**Edit:**
- `internal/orchcli#orchCLI.underAdoptWatch` -> `func (c *orchCLI) underAdoptWatch() (func(slug string) bool, string, error)`
- `internal/orchcli#orchCLI.watchCmd`
- `internal/orchcli/doc.go`
- `internal/orchengine#OwnerWatch`
- `internal/orchengine/doc.go`
- `internal/beamcli#hubDobbyDeps`
- `internal/beamcli#ownerArchiveBeforeRefusal`
- `internal/beamcli/doc.go`
- `internal/beamcli#TestHubDobbyDeps_ABeamWithNoOwnerOfItsOwnIsUnderARootAdopt`
- `internal/beamcli#TestOwnerArchiveBeforeRefusal_SkipsWhileAnAdoptCoversTheWorktree`
- `internal/dobbyengine#Deps`
- `internal/dobbyengine/handover.go`
- `internal/dobbyengine/doc.go`
- `cmd/lym#testOnlyCodeAllowed`

**Delete:**
- `internal/orchcli#UnderAdoptWatch`
- `internal/orchcli#TestUnderAdoptWatch_CoversThePrimeWhileARootAdoptIsUnfinished`
- `internal/orchcli#TestUnderAdoptWatch_CoversABeamWithNoOwnerOfItsOwnWhileARootAdoptIsUnfinished`

**Uses:**
- `internal/boardengine#Owners.AdoptSubject`
- `internal/boardengine#Registry.ReadsAsHere`
- `internal/boardengine#ReadOwners`
- `internal/boardengine#WriteHold`

**Intent:**
The dobby's hand-over and the orch watcher stop asking the adopt record whether an adopt in flight here covers their worktree; they read the Bolt hold.

`UnderOwnHold` returns the accessor: for a worktree slug (empty for the prime) it reads the hub's local Bolt through `ReadOwners` and this machine's identity through `machinename.ReadIdentity`.
It resolves the worktree's subject, `root` for the prime and `Owners.AdoptSubject(slug, hubgeom.ParentOf)` for a beam, so a beam with its own owner is its own subject.
It reports true only when a hold whose machine `Registry.ReadsAsHere` lists that subject among its subjects.
A foreign-owned beam outside the hold's subjects, such as a nested beam under an adopted root that keeps its own owner, is not covered, so its dobby hands over as it did under the adopt record.
A board or identity that does not read counts as no hold, logged once per episode (`logger.Episode`), as the record reader did.

Callers move to it in this card: `orchCLI.underAdoptWatch` returns the accessor and the slug; `watchCmd` builds it lazily as before and calls it in `OwnerWatch.UnderAdopt`;
`hubDobbyDeps` fills `UnderAdopt` with `UnderOwnHold(prime)(geom.Slug)`;
`ownerArchiveBeforeRefusal` skips the archive while `UnderOwnHold(prime)(slug)` holds.
The `Deps.UnderAdopt` and `OwnerWatch.UnderAdopt` field comments, handover.go's file comment and `probeOwner`'s comment and its `adopt hold:` log line, and the doc.go files of dobbyengine, orchengine, orchcli and beamcli say the seam reads a hold this machine wrote in Bolt over the worktree's subject, standing until the adopt's check is green.

Deleting `UnderAdoptWatch` leaves `internal/adoptrecord`'s watch with no caller a main reaches while `adoptcli` still imports the package, so `cmd/lym`'s `testOnlyCodeAllowed` gains `internal/adoptrecord.NewWatch`, `internal/adoptrecord.Watch.Covers`, `internal/adoptrecord.Watch.covers` and `internal/adoptrecord.Record.Covers`, whose only caller is `Watch.covers`, each Why saying card 9 removes the package, as the `internal/runfsm.Order` entry does.
The exact keys are the ones the guard reports at this card's tree.

`TestUnderOwnHold` (tier 1, over a hubforge hub; the test's own identity is the one envkit's machine file gives, and "another machine" is a name the registry entry of a second id holds; holds planted with `WriteHold`) rows:
`AHoldOfThisMachineCoversThePrime`, `AHoldOfThisMachineCoversABeamSubject`, `ABeamOutsideTheHoldsSubjectsIsNotCovered`, `AnotherMachinesHoldCoversNothing`, `NoHoldCoversNothing`, `AnUndecodableBoardCoversNothing`.
`TestHubDobbyDeps_ABeamWithNoOwnerOfItsOwnIsUnderARootAdopt` (tier 1) plants this machine's hold over root in place of a running adopt record and expects `UnderAdopt` true, and false with no hold.
`TestOwnerArchiveBeforeRefusal_SkipsWhileAnAdoptCoversTheWorktree` (tier 1) plants the hold the same way.

**ImpactSummary:** three readers in orchcli and beamcli switch from the adopt record to the Bolt hold; dobbyengine and orchengine keep their seams and change only comments and one log line; adoptrecord's watch gains test-only-code allowlist entries until card 9 removes the package.

**Commit:** 5: Read this machine's Bolt hold where the watcher and the dobby read the adopt record
