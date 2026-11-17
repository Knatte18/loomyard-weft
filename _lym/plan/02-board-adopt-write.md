# Card 2 — One-commit adopt write and the hold setters

**Create:**
- `plan:internal/boardengine#AdoptRequest` -> `type AdoptRequest struct`
- `plan:internal/boardengine#AdoptWrite` -> `type AdoptWrite struct`
- `plan:internal/boardengine#HoldRefusal` -> `type HoldRefusal struct`
- `plan:internal/boardengine#HoldRefusal.Error` -> `func (e *HoldRefusal) Error() string`
- `plan:internal/boardengine#WriteAdopt` -> `func WriteAdopt(boardDir string, req AdoptRequest) (AdoptWrite, error)`
- `plan:internal/boardengine#FixHold` -> `func FixHold(boardDir string, here machinename.Identity, fixTask string, tries map[string]int) (Hold, error)`
- `plan:internal/boardengine#ClearHold` -> `func ClearHold(boardDir string, here machinename.Identity) (bool, error)`
- `plan:internal/boardengine#TestWriteAdopt` -> `func TestWriteAdopt(t *testing.T)`

**Edit:**
- `internal/boardengine#commitBoardWriteAndPush` -> `func commitBoardWriteAndPush(boardDir, message string, write func() ([]string, error), beforePush func()) error`
- `internal/boardengine#WriteHold`
- `internal/boardengine#RegisterMachine`

**Extends:**
- `internal/boardengine/doc.go`

**Uses:**
- `internal/boardengine#Registry.sameMachine`
- `internal/boardengine#Registry.ReadsAsHere`
- `internal/boardengine#ownerFixture`
- `internal/boardengine#newOwnerFixture`
- `internal/boardengine#ownerFixture.pushFromOther`
- `internal/boardengine#ownerFixture.originOwners`
- `internal/boardengine#FabricReplay`

**Intent:**
D6 step 1 and `--take-hold` as one pull-first Bolt commit, plus the setters the check's ends call.

`commitBoardWriteAndPush` gains `beforePush`, run once between the commit and the first push, nil running nothing, as `OwnerRequest.BeforePush` does; `WriteHold` and `RegisterMachine` pass nil.

`AdoptRequest` carries `Machine`, `Identity` (registered in the same commit, as `OwnerRequest.Identity` does, refused through `Registry.Check` with its `*RegistrationError` and way forward), `Slugs` (the beam subjects), `Prime` (root is a subject), `TakeHold`, `ParentOf`, `Now` and `BeforePush` (nil in production, handed to `commitBoardWriteAndPush`).
`AdoptWrite` returns the `Hold` that stands after the push, `Kept` (this machine's same-set hold stood as it was), `Recorded` (the subjects whose owner reads as this machine after the push, `OwnerWritePrime` for root).

`WriteAdopt` runs its write through `commitBoardWriteAndPush`. Inside the write, over the store the pull brought, compared through `Registry.sameMachine` and `ReadsAsHere`:
1. No hold stands: append an `adopt` hold naming `Machine`, `At` from `Now`, and one `HoldSubject` per subject whose `PriorOwner` is the subject's effective owner before this write (prime owner for root).
2. This machine's hold over the same subject set (order ignored, root as `RootSubject`): keep it untouched, its kind, fix task and try counts included; write only owners and the registry; `Kept` is true.
3. This machine's hold over any other set, overlapping included: refuse with a `*HoldRefusal` naming the hold's subjects; way forward `lym adopt <hold's subjects>` first.
4. Another machine's hold, without `TakeHold`: refuse with a `*HoldRefusal` naming the writer machine and the hold's time; way forward: wait for that machine's adopt to finish, and if that machine has stopped, ask the operator. The text never spells the take-hold flag, since an agent may read it.
5. Another machine's hold with `TakeHold` over another set: refuse naming the hold's machine and subjects; way forward `lym adopt --take-hold <hold's subjects>`. Only the operator reaches this refusal, since the shell guard (card 4) denies the flag to every agent, so no agent reads the flag here.
6. Another machine's hold with `TakeHold` over the same set: rewrite the hold to name `Machine`, keeping kind, fix task, try counts and each `PriorOwner`.
Then it writes `Machine` as owner of each beam subject and as prime owner when root is a subject (the takeover path of `writeOwners`), and registers the identity.
A rejected push replays through `FabricReplay` (card 1), which refuses the commit whole over another machine's new hold and replays it otherwise.
After the push it re-reads the local Bolt: an owner of a subject that reads as another machine, or a hold another machine wrote, refuses with a `*HoldRefusal` naming that machine (the winner), way forward as in case 4; otherwise it returns the hold and `Recorded`.
An unreachable origin returns `ErrOwnerOriginUnreachable` wrapped, as the owner write does.
Bound of `TakeHold`: it moves only a hold over the same subject set to this machine; it skips no check and lets no run go on.

`FixHold` rewrites this machine's hold to kind `adopt-fix` naming `fixTask`, setting each subject's `Tries` from `tries` (subjects absent from the map keep theirs).
No hold standing refuses with a `*HoldRefusal`, way forward `lym adopt <subjects of tries>`, which writes a hold and checks again.
A hold of another machine refuses naming that machine: it took the hold over, so this machine's check ends here; way forward: wait for that machine's adopt, or ask the operator.
`ClearHold` removes this machine's hold and reports whether one stood; another machine's hold is left and reported false.
Both use `commitBoardWriteAndPush`, so each lands pushed or returns its error.

The doc.go names the three writers beside `WriteHold` as the only writers of `holds`.

`TestWriteAdopt` (tier 1, over `newOwnerFixture`'s board with a second clone) rows:
`NoHoldWritesTheHoldTheOwnersAndTheRegistryInOneCommit` (one new origin commit; hold subjects carry the prior owners),
`ThisMachinesSameSetHoldStaysWithItsKindFixTaskAndTries`,
`ThisMachinesOtherSetHoldRefusesNamingItsSubjects` (also an overlapping set),
`AnotherMachinesHoldRefusesNamingItsMachineAndTimeAndNoTakeHoldFlag`,
`TakeHoldRewritesTheHoldToThisMachineKeepingKindTaskTriesAndPriorOwners`,
`TakeHoldOverAnotherSetRefusesNamingTheHoldsSubjectsAndTheTakeHoldRerun`,
`TwoClonesAdoptingTheSameBeamEndWithOneOwnerAndTheLoserRefusesNamingTheWinner` (this clone's `BeforePush` runs the second clone's `WriteAdopt` of the same beam, which pushes first; this clone's push is rejected, the replay refuses whole, origin holds one owner and one hold, and the refusal names the second clone's machine),
`ARejectedPushMeetingNoNewHoldReplaysAndLands` (this clone's `BeforePush` pushes an unrelated entry change from the second clone; the adopt lands with its hold and owners),
`ARefusedRegistrationWritesNothing`, `UnreachableOriginIsErrOwnerOriginUnreachable`,
`FixHoldMakesTheHoldAdoptFixWithTaskAndTriesOnOrigin`, `FixHoldWithNoHoldRefuses`, `FixHoldOverAnotherMachinesHoldRefusesNamingIt`,
`ClearHoldRemovesTheOwnHold`, and `ClearHoldLeavesAnotherMachinesHold`.

**ImpactSummary:** new writers plus a `beforePush` parameter on the shared commit helper, whose two other callers pass nil; WriteHold, RegisterMachine and SetOwners behave as before.

**Commit:** 2: Add the one-commit adopt write and the hold setters to boardengine
