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
- `plan:internal/loomcli#TestParentPush` -> `func TestParentPush(t *testing.T)`

**Uses:**
- `internal/loomcli/doc.go`
- `internal/beamcli/doc.go`
- `plan:internal/fabriclayout#PushFailureHeldBack`
- `plan:internal/fabricengine#PushPrimeCodeTip`
- `plan:internal/fabricengine#PrimeCodeRefusalStands`
- `plan:internal/fabricengine#ClearPrimeCodeRefusal`
- `plan:internal/fabricengine#LandingCommits`
- `plan:internal/fabricengine#ResetParentToOrigin`
- `plan:internal/fabricengine#KeepHeldRecords`
- `internal/fabricengine#RecordPushFailure`
- `internal/fabricengine#OriginParentHoldsTaskWork`
- `internal/boardengine#PullBolt`
- `internal/pairteardown#HandOverOwnedElsewhere`
- `internal/pairteardown#RunPush`
- `internal/pairteardown#PushRun`
- `plan:internal/pushledger#OwedTips`
- `plan:internal/pushledger#Landing`
- `plan:internal/pushledger#Update`

**Intent:** D5: the parent target's pusher pushes a landing's parent branch, settles each landing once origin's parent holds it, and holds back what PATTERN-owner-machine forbids.

The parent target is the landing target for the prime's code branch and the beam pair's records target for a beam parent (D1); `ParentTarget` picks it from the run's parent pair, the landing target for the prime and the beam pair's target otherwise.
`RecordLanding` adds a landing to the parent target's entry with the push-now mark and starts or wakes that pusher; card 12 calls it from Finalize. A landing onto a beam parent writes no landing-key entry.
It writes the entry's `Worktree`, the parent's code worktree, and for a beam parent its `Slug`, the beam's slug, so `PruneLedger` (card 6) and `HeldRecordsCommit` read the entry like any pair's.
`settleParent` runs on any parent target whose entry holds landings, or whose parent branch carries a landing commit past origin that no landing records, that the entry's `Dropped` does not name, and whose run's board entry is not `done`.
`NewDeps` widens the landing target's `Ahead` to the same two reads with the same skips, so a hub whose ledger was lost still starts the landing target's pusher,
and the commit a pushes-off landing leaves, whose run Finalize marked `done` itself (card 12), starts no endless pusher.
`Deps` gains `Parent ParentDeps`, the seams it drives: `Fetch`, `OriginHolds(tip)`, `PushTip(tip)`, `ParentOwner()`, `RunOwner(slug)`, `WorkUnderOtherCommit(slug)`, `LandingCommits()`, `RunPastFinalize(slug)`, `ResetToOrigin()`, `HandOver(slug, owner)`, `DoneWrite(slug)`, `WakePair(slug)`, `RecordRefusal(reason, wayForward string)` and `ClearRefusal()`.
A nil `Parent.PushTip`, as a hand-built `Deps` or a task pair's records target leaves it, runs no parent step.

Every pass, and every `SettleRefused`, first brings the owners up from origin, before the re-record and before any owner read:
the landing target pulls the Bolt through `boardengine.PullBolt`, and a beam parent runs the beam pair's run-scope reconcile.
Every owner read of steps 1 to 4, the one step 2's hand-over depends on included, and every board-entry read of the re-record and the skips reads that pulled state, so no verdict predates an absence on origin (D5, PATTERN-owner-machine).
A pull or reconcile that cannot reach origin ends the pass owed, retried at D2's interval, with every landing standing and nothing written.

The ledger is a cache of the landings, which git rebuilds (PATTERN-lym-files):
each pass, before step 1, re-records through `RecordLanding` every landing commit `LandingCommits` finds past origin whose run has no landing, that the parent entry's `Dropped` does not name and whose board entry is not `done`, when `RunPastFinalize` reads the run past Finalize.
The re-recorded landing holds the oldest commit of the run's pair records branch past its tracking ref, so nothing card 10 kept back, Finalize's records commit included, goes out before the settle (card 9);
it holds none when no records commit is past the tracking ref, or when the pair no longer stands here.
`RunPastFinalize` reads the run's status file in its task worktree as done, or the pair as gone from this machine.

One pass of `settleParent`, in this order:
1. Fetch, then re-read each standing landing's `Tip` as the local parent commit `LandingCommits` names for its run, when that differs, as after a rebase in the parent's worktree,
   and remove from the entry's `Dropped` every run whose commit `LandingCommits` no longer lists;
   when that removes the last run `Dropped` names whose commit the parent's tip carried, clear the refused-push record step 2's rebase refusal wrote, so a rebase that followed its way forward unrefuses the target with no newer landing.
   Settle every landing whose `Tip` origin's parent holds, oldest first, however it got there: read its run's owner from the pulled state;
   for a run this machine owns, make its `done` write, remove its landing (which releases its pair hold), set the run's pair target's push-now mark through `WakePair`, whose entry write sets `Worktree` and `Slug` as every records-entry write does, and clear the parent's refused-push record.
   A run read as owned elsewhere gets no `done` write, which is its owner's: its landing is removed, its `RecordsCommit` moved into the run's pair target entry as `HeldRecords` and kept under the pair's held-records ref as step 2's drop does,
   and the run handed over through `pairteardown.HandOverOwnedElsewhere`; no refused-push record is written, so later landings on the target still push;
   a `RunOwner` read that fails leaves the landing standing for the next pass.
   A newer tip that landed settles every older landing with it, each through the same owner read.
2. For each standing landing whose work origin's parent holds under another commit (`OriginParentHoldsTaskWork`), drop it: remove it with no `done` write, name its run and landing commit in the parent entry's `Dropped`,
   move its `RecordsCommit` into the run's pair target entry as `HeldRecords` and keep it under the pair's held-records ref through `fabricengine.KeepHeldRecords` (card 10), so the hold outlives the landing and a lost ledger until the pair goes,
   and clear the parent's refused-push record.
   The drop sets that pair entry's `Slug`, the run's slug, and `Worktree`, the pair's code worktree, whether it creates the entry or finds one.
   When the landing commit alone puts the parent ahead, `ResetToOrigin`, through `fabricengine.ResetParentToOrigin`, runs `git reset --keep origin/<parent>`, for a run this machine owns and, before its hand-over, for a run owned elsewhere.
   A dropped run owned elsewhere is then handed over through `pairteardown.HandOverOwnedElsewhere`, whose pushes stop short of the held commit (card 9); a dropped run this machine owns keeps its pair until `lym fabric remove`.
   With other local commits on the landing commit no reset is made: the push is refused (D7) naming the commit, with the way forward a rebase in the parent's worktree that drops it, then the parent target's push;
   a push whose tip still carries a commit `Dropped` names stays refused this way on every later pass.
3. Read the parent's owner and the owner of every run whose landing the push would carry, every landing up to the tip it pushes, from the state pulled before the re-record.
   An owner read that fails is an owed push, retried at D2's interval.
4. A parent owned elsewhere refuses the push, naming the owning machine, with the way forward `lym adopt root` for the prime or `lym adopt <beam>` for a beam parent on this machine, then the parent target's push: `git push` in the prime's worktree, or `lym fabric push` for a beam parent.
   A run owned elsewhere refuses likewise, naming the run and `lym adopt` of its subject, then that push, or leaving the run to its owner, and holds back every recorded tip that carries its merge; an older landing's tip, which does not, is still pushed and settled on the prime. A beam parent's push carries every landing, so it is held back whole.
   A tip whose first-parent chain past origin carries a landing commit `LandingCommits` names for a run no landing records is refused the same way only while `RunPastFinalize` reads that run still at Finalize, naming the commit, with the way forward `lym loom resume` in that run's task worktree, which records its landing (card 12), or a rebase in the parent's worktree that drops the commit, then the parent target's push.
   A run past Finalize was re-recorded before step 1, and a landing commit of a run the parent entry's `Dropped` names is skipped here, since step 2's rebase refusal already names it.
   Each refusal writes one `held_back` record of the parent's branch through `RecordRefusal` and leaves every landing standing.
5. Push the newest tip not held back through `PushTip`: `fabricengine.PushPrimeCodeTip` for the prime, and for a beam parent the beam pair's `pairteardown.PushRun` with `RunPush.CodeCeiling` set to that tip (card 9). An owed failure returns owed, and a diverged origin is card 10's refusal. Once it lands, settle as in step 1.
Bound: no pass pushes a branch the operator made outside lym, nor a prime commit no landing recorded, and nothing moves while another machine owns the parent or a carried run.

`TargetRefused` reads the landing target as refused through `fabricengine.PrimeCodeRefusalStands`, so a refusal of the prime's code branch counts against the landing target alone (D3).
A refused parent target starts no pusher on any tick (card 6), which holds the notice to one.
The dobby tick settles it instead, in-process: `restartHubPushers` runs `SettleRefused` for the landing target and for each pair target holding landings or `Dropped` while `TargetRefused` reads it refused, so a beam parent whose only landing was dropped still meets step 1's clearing after the rebase.
`SettleRefused` pulls the owners and re-records as a pass does, then runs steps 1 and 2 alone and pushes nothing, so a tip the way forward's hand push put on origin settles with its `done` write, a landing whose run its owner landed under another commit is dropped, and either clears the record;
a rebase that removed the last commit `Dropped` names clears the drop's refusal through step 1 the same way, and a landing on origin whose run reads owned elsewhere is released and handed over here too, with no `done` write.
The next tick then starts the pusher of a target still owed.
It also clears a step-4 `held_back` record once every landing commit that record named is recorded or gone, as when a pass read the gap between Finalize's merge and its `RecordLanding`, so a routine landing that meets the gap is never left refused.
`pusherDeps` wires the seams for the landing target and for every beam pair target, one holding no landing included, so a beam's ordinary pass meets the parent steps for a landing commit no entry records yet.

`TestParentPush`, tier1, in `internal/loomcli` beside `pusherDeps`, over a `fabrickit` hub fixture with a bare origin and the ledger under the hub, adding about twenty seconds to one tier1 run.
It drives the pass through `pusherDeps`' real wiring: `fabricengine`'s parent steps, `boardengine`'s `done` write and `fabricengine.RecordPushFailure`'s refusal records.
An unreachable origin is the fixture's origin URL pointed at a path that does not exist, by `git remote set-url` run through `gitkit.Git`, and another machine's ownership is a takeover entry written through `takeoverkit`.
It pins:
two landings, each `done` write and pair release made only once the newer tip lands, and neither before;
the `done` write after the parent push and the hold released after it;
the landing target pulling the Bolt before the re-record and every owner read, in a pass and in `SettleRefused`;
a landing whose tip origin already holds, its run's takeover entry on origin's board only: neither a pass nor `SettleRefused` makes the `done` write; the landing is removed, its records commit kept as the pair entry's `HeldRecords`, the run handed over, and no refused-push record written;
a dropped landing with only a plain local commit on its landing commit refused naming a rebase, on the landing target and on a beam parent whose only landing it was; after the rebase drops the commit, `SettleRefused` clears the record, and a later landing's pusher starts and pushes;
a landing whose tip origin already holds settled on the next pass with its `done` write and its refused-push record cleared;
the prime owned by another machine: nothing pushed, no `done` write, one `held_back` record naming `lym adopt root`, then `git push` in the prime's worktree;
that way forward run, the tip on origin: `SettleRefused` makes the `done` write and clears the record;
a beam run landed on the prime and adopted by another machine: no parent push, no `done` write, the refusal naming `lym adopt` of that run, and its owner's landing under another commit dropped by `SettleRefused`, the record cleared;
a beam parent: origin's beam branch ends at the newest tip, and a carried run owned elsewhere holds back the whole push;
a landing whose work origin holds under another commit, its pair entry already written by an earlier push, dropped with no push and no `done` write, its records commit kept as that entry's `HeldRecords`, the entry's `Slug` and `Worktree` written, and the run named in the parent entry's `Dropped`;
an owned-elsewhere drop whose landing commit alone puts the parent ahead leaving the local parent at origin's tip, and the run handed over;
a dropped run owned here with only its landing commit ahead reset to origin, and with a newer landing on it the push refused naming a rebase, after which the rebased tip pushed by hand settles through `SettleRefused` reading the new tip from `LandingCommits`;
that dropped run with a newer landing on it, its pair then removed through `lym fabric remove`: later passes neither re-record nor push its commit;
the ledger directory deleted after a drop: a pair push still stops short of the held commit;
a tip carrying a landing commit of a run still at Finalize that no landing records, refused naming the commit and `lym loom resume`, and once `RecordLanding` records it, the record cleared by `SettleRefused`;
the same commit of a done run re-recorded and pushed, with its `done` write after the push;
a landing commit whose run's board entry is `done`, as a pushes-off landing leaves it, read with pushes on: the landing target's `Ahead` reads false and nothing is re-recorded;
the ledger directory deleted after a landing and the run's done transition committed: the landing target's `Ahead` reads true over the empty ledger, the next pass re-records the landing, origin still lacks Finalize's records commit after the re-record, and the `done` write comes only after the parent push lands;
an owner read that cannot reach origin waiting one interval with no refusal;
and `RecordLanding` over `ParentTarget` of a beam parent writing the landing on the beam pair's key with the beam's `Slug` and `Worktree`, and no landing-key entry.
`TestStart` gains a row: a landing target holding landings with its record standing spawns nothing.
`TestRestartHubPushers` gains rows: a refused landing target is handed to `SettleRefused` and to no start, and so is a refused beam pair target holding only `Dropped`.

**ImpactSummary:** Adds the parent step to the pusher's pass for targets that hold landings or carry an unrecorded landing commit, and a refused-target settle to the dobby tick; a target with no landing pushes as before.
