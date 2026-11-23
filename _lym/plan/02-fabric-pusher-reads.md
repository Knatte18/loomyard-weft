# Card 2 — fabric-pusher-reads

**Create:**
- `plan:internal/fabricengine#RunCommitsAhead` -> `func RunCommitsAhead(l *lymcwd.Location) (bool, error)`
- `plan:internal/fabricengine#RefusedPushStands` -> `func RefusedPushStands(l *lymcwd.Location) (bool, error)`
- `plan:internal/fabricengine#TipsOnOrigin` -> `func TipsOnOrigin(l *lymcwd.Location) (bool, error)`
- `plan:internal/fabricengine_test#TestRunCommitsAhead` -> `func TestRunCommitsAhead(t *testing.T)`
- `plan:internal/fabricengine_test#TestRefusedPushStands` -> `func TestRefusedPushStands(t *testing.T)`
- `plan:internal/fabricengine_test#TestTipsOnOrigin` -> `func TestTipsOnOrigin(t *testing.T)`

**Extends:**
- `internal/fabricengine/doc.go`

**Uses:**
- `internal/fabricengine#ListFailedPushes`
- `internal/fabricengine#IsPrimeWorktree`
- `internal/fabricengine#RecordsWorktree`
- `internal/fabricengine#HeadContains`

**Intent:**
Add the three local reads the pusher's start, the dobby's tick and Worktree-Teardown judge by.
Each reads only local refs, never origin, so it makes no network call.
The worktree `l` names is the hub's prime or a task pair's code worktree, its records worktree from `RecordsWorktree`.

`RunCommitsAhead` answers D5's "ahead of origin":
- A branch is ahead when it holds a commit its origin tracking ref lacks,
  or when it has no upstream configured and holds at least one commit.
  A branch whose upstream is set but whose tracking ref is absent, as after a fetch pruned it, reads level:
  origin ended it, and it is left to its teardown.
- The prime is ahead when its records branch is; its code branch is the operator's and never counts.
- A pair is ahead when its records branch is, or when its code branch holds the code tip
  that the `Warp-SHA` trailer of its local records tip names and that branch's tracking ref lacks that tip.
  A code branch with no upstream lacks every tip.
  Code commits past the named tip are not counted: the transition that names them starts its own pusher.
  A records tip with no `Warp-SHA` trailer counts no code side.

`RefusedPushStands` reports whether a refused-push record stands for the records branch of `l`, or, for a pair, its code branch.
A refused-push record is a failed-push record, read through `ListFailedPushes`, of a class other than transport, deferred and teardown_lock.
An owed-class record any caller wrote reads false.
Records are keyed by repo side and branch, so a refused prime records push reads true for every batten run on this machine.

`TipsOnOrigin` reports whether origin's tracking refs hold the local records tip and, for a pair, the local code tip of `l`:
each tracking ref is the local tip or a descendant of it.
A branch with no upstream fails it.
A branch whose upstream is set and whose tracking ref is absent reads held, as `RunCommitsAhead` reads it level:
origin ended it, so its teardown has nothing to push.
Worktree-Teardown removes a pair only while it holds (card 10).

`fabricengine/doc.go` gains one line naming the three reads.

Every test is tier1 in the external `fabricengine_test` package over a hub fixture with a bare origin.
- `TestRunCommitsAhead` rows: a prime records commit unpushed reads ahead; a level prime reads level;
  a prime code commit alone reads level; a records branch with no upstream and one commit reads ahead;
  a records branch whose tracking ref was deleted with its upstream set reads level;
  a pair whose records branch is level and whose code branch holds the named tip origin lacks reads ahead;
  a pair whose code branch is ahead only by commits past the named tip reads level;
  a pair whose code branch has no upstream and holds the named tip reads ahead;
  a pair whose level records tip carries no `Warp-SHA` trailer, with an unpushed code commit, reads level.
- `TestRefusedPushStands` rows: no record; a not_fast_forward record of the records branch; one of the pair's code branch;
  an owed-class record (transport) of the records branch reads false; a record of another branch reads false.
- `TestTipsOnOrigin` rows: both tips pushed; a records commit unpushed; a code commit unpushed; a code branch with no upstream;
  a code branch whose upstream is set and whose tracking ref was deleted reads held.

**ImpactSummary:** Three new read-only functions in fabricengine and one doc line; no existing caller changes.
