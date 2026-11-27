# Card 10 — parent-push-git

**Edit:**
- `internal/fabricengine#FailedPushWayForward`
- `internal/fabricengine#pushStep`

**Create:**
- `plan:internal/fabriclayout#PushFailureHeldBack` -> `const PushFailureHeldBack PushFailureClass = "held_back"`
- `plan:internal/fabriclayout#LandingTrailerKey` -> `const LandingTrailerKey = "Lym-Landing"`
- `plan:internal/fabricengine#PushPrimeCodeTip` -> `func PushPrimeCodeTip(prime *lymcwd.Location, tip string, opts SyncOptions) error`
- `plan:internal/fabricengine#PrimeCodeRefusalStands` -> `func PrimeCodeRefusalStands(prime *lymcwd.Location) (bool, error)`
- `plan:internal/fabricengine#ClearPrimeCodeRefusal` -> `func ClearPrimeCodeRefusal(prime *lymcwd.Location) error`
- `plan:internal/fabricengine#LandingCommits` -> `func LandingCommits(l *lymcwd.Location) (map[string]string, error)`
- `plan:internal/fabricengine#AheadByOnly` -> `func AheadByOnly(l *lymcwd.Location, tips []string) (bool, error)`
- `plan:internal/fabricengine#ResetParentToOrigin` -> `func ResetParentToOrigin(l *lymcwd.Location) error`
- `plan:internal/fabricengine_test#TestPushPrimeCodeTip` -> `func TestPushPrimeCodeTip(t *testing.T)`
- `plan:internal/fabricengine_test#TestParentAheadReads` -> `func TestParentAheadReads(t *testing.T)`
- `plan:internal/fabricengine#TestFailedPushWayForward` -> `func TestFailedPushWayForward(t *testing.T)`

**Uses:**
- `internal/fabricengine/doc.go`
- `internal/fabriclayout/doc.go`
- `internal/fabriclayout#PushFailureClass`
- `internal/fabricengine#RecordPushFailure`
- `internal/fabricengine#ClearPushFailure`
- `internal/fabricengine#RefusedPushStands`

**Intent:** The git steps of D5's parent targets, which cards 11, 12 and 16 call; every git run on a parent stays in `internal/fabricengine`, per PATTERN-fabric-boundary.

`PushPrimeCodeTip` pushes the prime's code branch to `tip`, fast-forward only and never forced, through `pushClassified` with no transport backoff of its own (D2), under the remote-refs lock.
It pushes nothing past `tip`, so a commit on the prime's code branch that no landing recorded never rides along (Scope, Out).
A transport failure returns an owed `*PushFailedError` and writes no record when `opts` asks so, as the pusher does.
A rejection that stays after the takeover read and the rebase-free retry is a refusal (D7): it records one failed-push record of the prime's code branch, class `not_fast_forward`,
whose way forward names the landing commits origin lacks and the merge of origin's parent into the local parent in the prime's worktree, then `git push` there (`codeDivergedWayForward` with `prime` set).

`LandingCommits` lists, for the parent branch checked out at `l`, each first-parent commit past its tracking ref that carries the `Lym-Landing` trailer, keyed by the run the trailer names; card 12 writes the trailer.
`AheadByOnly` reports whether every first-parent commit between the tracking ref and the local head is one of `tips`, so a parent ahead of origin by an owed tip and an unrecorded commit below it is not read as ahead by owed tips alone (D5).
`ResetParentToOrigin` runs `git reset --keep origin/<branch>` in the parent's code worktree, the reset D5 names for a dropped landing whose commit alone puts the parent ahead.

`PushFailureHeldBack` is the class of a parent push D5 holds back, written by card 11 through `RecordPushFailure`: a parent or a carried run another machine owns, or a tip carrying a landing commit no entry records.
Its way forward is the record's own `WayForward`, as `anchor_change`'s is, so `FailedPushWayForward` returns it, and `pushStep` names `git push` in the prime's worktree for the prime's code branch and `lym fabric push` for a beam parent, as today.
`PrimeCodeRefusalStands` reports whether a refused-push record of the prime's code branch stands, which `RefusedPushStands` skips for a prime location:
the landing target reads its refusals through it, while a refusal of the prime's records branch keeps counting against the prime's records target alone (D3).
`ClearPrimeCodeRefusal` removes that record.

`TestPushPrimeCodeTip`, tier1, over a hub fixture: a landing tip one merge ahead of origin is pushed and origin's branch is that tip;
a local commit past `tip` stays off origin;
an unreachable origin returns an owed error and writes no record;
origin moved on by another clone is refused with one record whose way forward names the landing commit and `git push` in the prime's worktree, and `PrimeCodeRefusalStands` then reads true and `ClearPrimeCodeRefusal` clears it.
`TestParentAheadReads`, tier1, over a hub fixture: `LandingCommits` finds two trailer commits past origin and skips a plain commit between them;
`AheadByOnly` reads true for a head that is an owed tip one commit past origin and false with an unrecorded commit below it;
`ResetParentToOrigin` puts the local branch back on origin's tip.
`TestFailedPushWayForward`, tier0, adding under a second to one tier0 run, pins that a `held_back` record returns its own way forward and that its `pushStep` names `git push` for the prime's code branch and `lym fabric push` for a beam parent.

**ImpactSummary:** Adds one failure class, one trailer key and six parent-branch helpers; `FailedPushWayForward` gains one case and no existing class changes its text.
