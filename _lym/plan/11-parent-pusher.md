# Card 11 — parent-pusher

**Edit:**
- `internal/runpusher#Deps`
- `internal/runpusher#NewDeps`
- `internal/runpusher#pushUntilLevel`
- `internal/runpusher#TestStart`
- `internal/runpusher/doc.go`
- `internal/loomcli#pusherDeps`
- `internal/loomcli#TestPusherDeps`
- `plan:internal/beamcli#restartHubPushers`
- `plan:internal/beamcli#TestRestartHubPushers`
- `plan:internal/runpusher#TargetRefused`

**Create:**
- `plan:internal/runpusher#ParentDeps` -> `type ParentDeps struct`
- `plan:internal/runpusher#settleParent` -> `func settleParent(ctx context.Context, t Target, deps Deps) (pushed bool, err error)`
- `plan:internal/runpusher#SettleRefused` -> `func SettleRefused(ctx context.Context, t Target, deps Deps) error`
- `plan:internal/runpusher#RecordLanding` -> `func RecordLanding(parent Target, landing pushledger.Landing, now time.Time) error`
- `plan:internal/runpusher#ParentTarget` -> `func ParentTarget(prime, parent *lymcwd.Location) (Target, error)`
- `plan:internal/runpusher_test#TestParentPush` -> `func TestParentPush(t *testing.T)`

**Uses:**
- `internal/loomcli/doc.go`
- `internal/beamcli/doc.go`
- `plan:internal/fabriclayout#PushFailureHeldBack`
- `plan:internal/fabricengine#PushPrimeCodeTip`
- `plan:internal/fabricengine#PrimeCodeRefusalStands`
- `plan:internal/fabricengine#ClearPrimeCodeRefusal`
- `plan:internal/fabricengine#LandingCommits`
- `plan:internal/fabricengine#ResetParentToOrigin`
- `internal/fabricengine#RecordPushFailure`
- `internal/fabricengine#OriginParentHoldsTaskWork`
- `internal/boardengine#PullBolt`
- `internal/pairteardown#HandOverOwnedElsewhere`
- `plan:internal/pushledger#OwedTips`
- `plan:internal/pushledger#Landing`
- `plan:internal/pushledger#Update`

**Intent:** D5: the parent target's pusher pushes a landing's parent branch, settles each landing once origin's parent holds it, and holds back what PATTERN-owner-machine forbids.

The parent target is the landing target for the prime's code branch and the beam pair's records target for a beam parent (D1); `ParentTarget` picks it from the run's parent pair, the landing target for the prime and the beam pair's target otherwise.
`RecordLanding` adds a landing to the parent target's entry with the push-now mark and starts or wakes that pusher; card 12 calls it from Finalize. A landing onto a beam parent writes no landing-key entry.
`settleParent` runs on any target whose entry holds landings.
`Deps` gains `Parent ParentDeps`, the seams it drives: `Fetch`, `OriginHolds(tip)`, `PushTip(tip)`, `ParentOwner()`, `RunOwner(slug)`, `WorkUnderOtherCommit(slug)`, `LandingCommits()`, `ResetToOrigin()`, `HandOver(slug, owner)`, `DoneWrite(slug)`, `WakePair(slug)`, `RecordRefusal(reason, wayForward string)` and `ClearRefusal()`.
A nil `Parent.PushTip`, as a hand-built `Deps` or the records target of a pair with no landing leaves it, runs no parent step.

One pass of `settleParent`, in this order:
1. Fetch, then re-read each standing landing's `Tip` as the local parent commit `LandingCommits` names for its run, when that differs, as after a rebase in the parent's worktree.
   Settle every landing whose `Tip` origin's parent holds, oldest first, however it got there: read its run's owner again, make its `done` write, remove its landing (which releases its pair hold), set the run's pair target's push-now mark through `WakePair`, and clear the parent's refused-push record.
   A newer tip that landed settles every older landing with it.
2. For each standing landing whose work origin's parent holds under another commit (`OriginParentHoldsTaskWork`), drop it: remove it with no `done` write, move its `RecordsCommit` into the run's pair target entry as `HeldRecords`, so the hold outlives the landing until the pair goes, and clear the parent's refused-push record.
   A dropped run owned elsewhere is handed over through `pairteardown.HandOverOwnedElsewhere`, whose pushes stop short of the held commit (card 9) and which resets the local parent to origin's when the landing commit alone puts it ahead.
   A dropped run this machine owns keeps its pair until `lym fabric remove`; `ResetToOrigin`, through `fabricengine.ResetParentToOrigin`, runs `git reset --keep origin/<parent>` when the landing commit alone puts the parent ahead.
   With other local commits on the landing commit no reset is made: the push is refused (D7) naming the commit, with the way forward a rebase in the parent's worktree that drops it, then the parent target's push.
3. Read the parent's owner and the owner of every run whose landing the push would carry, every landing up to the tip it pushes; the landing target pulls the Bolt through `boardengine.PullBolt` first and reads every owner from it, and a beam parent reads them through the beam pair's run-scope reconcile.
   An owner read that cannot reach origin is an owed push, retried at D2's interval.
4. A parent owned elsewhere refuses the push, naming the owning machine, with the way forward `lym adopt root` for the prime or `lym adopt <beam>` for a beam parent on this machine, then the parent target's push: `git push` in the prime's worktree, or `lym fabric push` for a beam parent.
   A run owned elsewhere refuses likewise, naming the run and `lym adopt` of its subject, then that push, or leaving the run to its owner, and holds back every recorded tip that carries its merge; an older landing's tip, which does not, is still pushed and settled on the prime. A beam parent's push carries every landing, so it is held back whole.
   A tip whose first-parent chain past origin carries a landing commit `LandingCommits` names for a run no landing records is refused the same way, naming the commit, with the way forward `lym loom resume` in that run's task worktree, which records its landing (card 12), or a rebase in the parent's worktree that drops the commit, then the parent target's push.
   Each refusal writes one `held_back` record of the parent's branch through `RecordRefusal` and leaves every landing standing.
5. Push the newest tip not held back through `PushTip`: `fabricengine.PushPrimeCodeTip` for the prime, and for a beam parent the beam pair's `PushRun` with that tip as its code ceiling (card 9). An owed failure returns owed, and a diverged origin is card 10's refusal. Once it lands, settle as in step 1.
Bound: no pass pushes a branch the operator made outside lym, nor a prime commit no landing recorded, and nothing moves while another machine owns the parent or a carried run.

`TargetRefused` reads the landing target as refused through `fabricengine.PrimeCodeRefusalStands`, so a refusal of the prime's code branch counts against the landing target alone (D3).
A refused parent target starts no pusher on any tick (card 6), which holds the notice to one.
The dobby tick settles it instead, in-process: `restartHubPushers` runs `SettleRefused` for the landing target and for each pair target holding landings while `TargetRefused` reads it refused.
`SettleRefused` runs steps 1 and 2 alone and pushes nothing, so a tip the way forward's hand push put on origin settles with its `done` write, a landing whose run its owner landed under another commit is dropped, and either clears the record; the next tick then starts the pusher of a target still owed.
`pusherDeps` wires the seams for the landing target and every pair target.

`TestParentPush`, tier0, over a real ledger under `t.TempDir()` and fake `ParentDeps`, adding about a second to one tier0 run, pins:
two landings, each `done` write and pair release made only once the newer tip lands, and neither before;
the `done` write after the parent push and the hold released after it;
the landing target pulling the Bolt before its owner reads;
a landing whose tip origin already holds settled on the next pass with its `done` write and its refused-push record cleared;
the prime owned by another machine: nothing pushed, no `done` write, one `held_back` record naming `lym adopt root`, then `git push` in the prime's worktree;
that way forward run, the tip on origin: `SettleRefused` makes the `done` write and clears the record;
a beam run landed on the prime and adopted by another machine: no parent push, no `done` write, the refusal naming `lym adopt` of that run, and its owner's landing under another commit dropped by `SettleRefused`, the record cleared;
a beam parent: `PushTip` receives the newest tip, and a carried run owned elsewhere holds back the whole push;
a landing whose work origin holds under another commit dropped with no push and no `done` write and its records commit kept as the pair entry's `HeldRecords`, a run owned elsewhere handed over through the seam,
a dropped run owned here with only its landing commit ahead reset through `ResetToOrigin`, and with a newer landing on it the push refused naming a rebase, after which the rebased tip pushed by hand settles through `SettleRefused` reading the new tip from `LandingCommits`;
a tip carrying a landing commit no landing records refused naming the commit and `lym loom resume`;
an owner read that cannot reach origin waiting one interval with no refusal;
and `RecordLanding` over `ParentTarget` of a beam parent writing the landing on the beam pair's key and no landing-key entry.
`TestStart` gains a row: a landing target holding landings with its record standing spawns nothing.
`TestRestartHubPushers` gains a row: a refused landing target is handed to `SettleRefused` and to no start.

**ImpactSummary:** Adds the parent step to the pusher's pass for targets that hold landings and a refused-target settle to the dobby tick; a target with no landing pushes as before.
