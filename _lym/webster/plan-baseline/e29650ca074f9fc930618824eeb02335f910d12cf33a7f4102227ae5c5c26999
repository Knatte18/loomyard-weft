# Card 14 — prime-records-push-guard

**Create:**
- `plan:internal/fabricengine#PushFailureOperatorUnpushed` -> `const PushFailureOperatorUnpushed PushFailureClass = "operator_unpushed"`
- `plan:internal/fabricengine#primePushOperatorCommits` -> `func primePushOperatorCommits(prime, branch string) ([]string, error)`
- `plan:internal/fabricengine#primeRunsCarried` -> `func primeRunsCarried(prime, branch string) ([]string, error)`
- `plan:internal/fabricengine#holdPrimePushForOperator` -> `func holdPrimePushForOperator(hubPath, prime string, now time.Time) error`
- `plan:internal/fabricengine_test#TestPrimePushGuard` -> `func TestPrimePushGuard(t *testing.T)`

**Edit:**
- `internal/fabricengine#PushPairAnchored`
- `internal/fabricengine#coalescePushSides`
- `internal/fabricengine#retryWeftPush`
- `internal/fabricengine#takeoverBranch.land`
- `internal/fabricengine#IsPushOwed`
- `internal/fabricengine#FailedPushesOwedBy`
- `internal/fabricengine#FailedPushWayForward`
- `internal/runfsm#Inputs`
- `internal/runfsm#Verdict`
- `internal/runfsm#Act`
- `internal/runfsm#Verdict.Refusal`
- `internal/pairteardown#hubRunReader`
- `internal/fabricengine_test#TestIsPushOwed`
- `internal/fabricengine_test#TestFailedPushRecord`
- `internal/runfsm_test#TestAct`

**Extends:**
- `internal/fabricengine/doc.go`

**Uses:**
- `internal/runfsm/doc.go`
- `internal/pairteardown/doc.go`
- `internal/fabricengine#IsRecordsOnlyCommit`
- `internal/fabricengine#IsFabricMergeCommit`
- `internal/fabricengine#PushFailureProtected`
- `internal/fabricengine#oneRepositoryAt`
- `internal/hubforge#NewOneRepoHub`
- `internal/hubforge#AddPairWith`

**Intent:** D7's guarded records push of the prime, whose branch is also the operator's code branch.

`holdPrimePushForOperator(hubPath, prime, now)` is the guard every records push of a one-repository prime runs before it pushes:
`PushPairAnchored`, the detached push's `coalescePushSides` (reached from `Fabric.Commit`, `lym fabric sync` and every other `SpawnDetachedPush`), `retryWeftPush` before it pushes again after a replay,
and `takeoverBranch.land` before each push of a takeover entry on a one-repository prime, the parent `TakeOverRun` writes on when none is given.
On a hold `land` returns the guard's error joined with `dropEntry`'s, as its other push failures do, so the prime keeps no unpushed takeover entry and `TakeOverRun` reports the owed push.
It returns nil when `primePushOperatorCommits` returns none, and otherwise a `*PushFailedError` the caller returns without pushing.
`primePushOperatorCommits` lists the commits not reachable from origin's ref of the branch and keeps each that is neither records-only (`IsRecordsOnlyCommit`) nor a fabric merge commit (`IsFabricMergeCommit`)
nor reachable from a second parent of such a merge while some origin ref holds it (`git for-each-ref --contains <sha> refs/remotes/origin`).
A commit message alone admits nothing, so a code commit carrying the merge trailer stays an operator commit.
- With operator commits the guard pushes nothing and records a failed push of class `operator_unpushed` (`PushFailureOperatorUnpushed`) for the prime's branch,
  with `Commits` the operator commits and `Runs` from `primeRunsCarried`, the `Run-Transition` slugs of the unpushed commits; its notice is sent as a refused push's is.
  `FailedPushWayForward` names the operator's commits and `git push` in the prime, and, when origin refuses that push because its branch moved, `git merge origin/<branch>` first; the next records push then carries the records.
- Once a `protected` failed-push record exists for the prime's branch, the guard's refusal is recorded as `protected` instead, with `Commits` set;
  `FailedPushWayForward` of a `protected` record carrying `Commits` names both steps, allowing direct pushes and the operator's `git push` of those commits, so the line stays right once the protection is lifted.
- A push origin refuses as protected records `protected` through card 4's classification; the transition seam completes past it (card 11).
`IsPushOwed` reads a `*PushFailedError` of class `operator_unpushed` as owed and `FailedPushesOwedBy` lists such records, so a run in the prime waits at its transition, retrying, until the operator pushes.
The act rule's refusal names the record's own way forward: `runfsm.Inputs` and `runfsm.Verdict` gain `PushOwedWayForward`, which `Act` copies,
`pairteardown`'s `hubRunReader` fills it from `FailedPushWayForward` of the newest owed record, and `Verdict.Refusal` names it in place of `lym fabric push` when it is set.
Bound: a squash landing's commit and a fabric merge over hand-resolved code conflicts are not fabric merge commits, so the guard owes each as `operator_unpushed` until a landing's push of its parent or the operator's `git push` takes it to origin.
While the prime's branch refuses direct pushes its records stay unpushed and no other machine sees them.

Tests:
- `TestPrimePushGuard` (tier1, `NewOneRepoHub`; a records transition commit in the prime) rows:
  `OperatorCommitHoldsThePush` (an operator commit beneath the records commit: origin unmoved, an `operator_unpushed` record naming the commit and listing the prime run, one notice through a recording notifier, `IsPushOwed` true on the error);
  `DetachedPushOfThePrimeIsHeldToo` (the same state pushed through `CoalescePushBothAt` on the prime: origin unmoved and the same record);
  `FastForwardedOperatorCommitHoldsThePush` (the operator fast-forwarded the prime from a branch origin holds: same outcome);
  `CodeCommitCarryingTheMergeTrailerHoldsThePush`;
  `RecordsOnlyAndFabricMergeCommitsPush` (a fabric merge made before the prime's `.lym` was removed is still admitted);
  `ProtectedRefusalRecordsProtected` (pre-receive hook printing `GH006: Protected branch update failed`: a `protected` record; a following guard refusal over an operator commit is recorded `protected` too, its way forward naming both steps);
  `ProtectedRefusalOfATaskBranchKeepsTodaysClass` (a pair through `AddPairWith` whose branch the same hook refuses: the record's class is not `protected`, and `IsPushProtected` reads false);
  `TakeoverEntryPushOfThePrimeIsHeld` (`TakeOverRun` with an operator commit unpushed in the prime: origin's prime branch unmoved, an `operator_unpushed` record, and no takeover entry left in the run's status file).
- `TestIsPushOwed` (tier0) gains the row `OperatorUnpushed` (owed).
- `TestFailedPushRecord` (tier0): its row `OwedByListsTheRunsOwedClassesNewestFirst` gains an `operator_unpushed` record naming the run, which `FailedPushesOwedBy` lists;
  new rows assert the `operator_unpushed` way forward names the commits, `git push` and `git merge origin/<branch>`, and a `protected` record carrying `Commits` names both steps.
- `TestAct` (tier0) gains a push-owed row with `PushOwedWayForward` set: the refusal names it and not `lym fabric push`.

**ImpactSummary:** Every records push of a one-repository prime gains a guard; IsPushOwed and FailedPushesOwedBy read one more class as owed, which their callers in landingshed, loomcli, battencli, tendercli and pairteardown then hold as owed; the act rule's owed refusal names the record's way forward.
