---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: "_lym/plan/06-pusher-target-kinds.md#Intent"
    summary: "The board target card 7 now spawns for every board write never commits it. A board write only saves files and spawns the sync (internal/boardengine/board.go, `boardCriticalSection`), which commits and pushes. Card 6 reads the board's `Ahead` as `Bolt.HasUnpushed`, false with only uncommitted files, so the pusher reads level and exits (D1). When it does push, `Board.Sync` records and mails every owed failure through `Bolt.Push`, which is issue #238 for the board (D3)."
    fix: "In card 6, make the board's `Ahead`, in the pusher and in `starter.start` with `ifAhead`, true when the Bolt `HasUncommitted` or `HasUnpushed`, and only `HasUncommitted` when `boardengine.ApplySkipEnv` reads pushes off, so no pusher runs endlessly. Add under Create `plan:internal/boardengine#Board.PusherSync` -> `func (b *Board) PusherSync() error`, and under Uses `internal/fabricengine#Bolt.HasUncommitted`. It commits as `Sync` does and pushes with `OwedUnrecorded` and card 5's single attempt; `pusherDeps` uses it as the board's `Push`, with `ApplySkipEnv` applied. Add a tier1 row: a board write left uncommitted is committed and pushed by the board pusher, and over an unreachable origin it is committed, stays owed and writes no failed-push record."
  - id: F2
    severity: BLOCKING
    class: design
    location: "_lym/plan/10-parent-push-git.md#Intent"
    summary: "A local landing that does not squash (`landing.squash` false, or `beam.pull_request` `merge-in`) makes no commit. The parent-side merge runs `git merge --ff --no-commit` (internal/gitrepo/merge.go, `MergeStart`) and fast-forwards, since the task already merged the parent in. No `Lym-Landing` trailer exists and the first-parent chain runs through task commits. So the lost-ledger records hold, the beam code ceiling, the re-record and `LandingStands` see nothing, and `AheadByOnly` reads false for the recorded tip, which makes a second landing Stuck and the Land pull judge the prime ahead (D5, D13)."
    fix: "Make every local landing leave one trailer commit on the parent. In card 12, add under Edit `internal/fabriclayout#MergeOptions` and `internal/fabricengine#Fabric.Merge`; under Create `plan:internal/gitrepo#Repo.MergeStartNoFastForward` -> `func (r *Repo) MergeStartNoFastForward(ref string) (MergeOutcome, error)`; and under Uses `internal/fabricengine#TestMergeVocabulary_ResultAndOptionsFieldsAreSideFree`, `internal/fabriclayout/doc.go` and `internal/gitrepo/doc.go`. A new `MergeOptions` field, which Finalize sets, makes a non-squash parent-side merge a merge commit carrying the landing message. State it in the overview's trailer decision. Add a `TestLandingDeps_LocalLanding` row with squash off: the parent head is a merge commit with the trailer, and a second landing on it merges while the first push is owed."
  - id: F3
    severity: BLOCKING
    class: design
    location: "_lym/plan/11-parent-pusher.md#Intent"
    summary: "Step 1 reads the run's owner and makes the `done` write before step 3's `PullBolt`, and `SettleRefused` runs steps 1 and 2 only, so it never pulls. The verdict can predate an absence, against D5 (the landing target reads every owner from a Bolt it pulls first) and PATTERN-owner-machine. Step 1 also states no outcome for a run read as owned elsewhere or an owner read that fails."
    fix: "In card 11, move the owner source ahead of step 1: the landing target pulls the Bolt through `boardengine.PullBolt`, and a beam parent runs its run-scope reconcile, before any owner read of a pass or of `SettleRefused`, including the one step 2's hand-over depends on. In step 1, a run read as owned elsewhere gets no `done` write, keeps its landing and writes the `held_back` record naming `lym adopt` of its subject; an unreachable read leaves the landing for the next pass. Add a `TestParentPush` row: the tip is already on origin and the takeover is on origin's board only, and neither a pass nor `SettleRefused` writes `done`."
  - id: F4
    severity: BLOCKING
    class: design
    location: "_lym/plan/11-parent-pusher.md#Intent"
    summary: "The refusal of a drop with other commits on the dropped one is never cleared once its way forward is followed and no newer landing stands. A plain `git push` clears no lym record. Step 1 clears only on a settle, step 2 only on a drop, and `SettleRefused` only a step-4 record. The landing target then reads refused for good, so no tick or landing starts its pusher (D5)."
    fix: "In card 11 step 1, when it removes from `Dropped` the last run whose commit the parent's tip carried, clear the refused-push record that drop wrote, and state that `SettleRefused` does the same. Add a `TestParentPush` row: a dropped landing with only a plain commit on it is refused; after the rebase, `SettleRefused` clears the record, and a later landing's pusher starts and pushes."
  - id: F5
    severity: BLOCKING
    class: design
    location: "_lym/plan/20-dobby-push-summary.md#Intent"
    summary: "`Waits` claims every run the Go driver waits on but reads only pair wait records. Batten's own Go driver runs in the prime (internal/battencli/drive.go, `driverDepsFor`), and no card gives it `WaitRecordPath` or `NetworkWaitCeiling`. A Land `git-transport` stop (card 16) is then re-stepped every 15 s with no wait record, absent from the summary and not read as waiting (D3, D6)."
    fix: "In card 2, add `internal/battencli#battenCLI.driverDepsFor` under Edit and have it set `WaitRecordPath` to `shedrun.DriveWaitRecord` over the prime location and the slug, and `NetworkWaitCeiling` as `goDriverDepsFor` does. In card 3, have `DobbyRuns` also set `Waiting` and `WaitDied` from the batten run's own record and driver lock. In card 20, have `Waits` read each batten run's record in the prime as well as each pair's. Add rows: a batten run waiting at Land owes no run-watch line and is listed in the summary."
  - id: F6
    severity: BLOCKING
    class: design
    location: "_lym/plan/17-catch-up-hands-to-pusher.md#Intent"
    summary: "`TestPushOwedForCatchUp`'s unreachable-prime row runs the real push, whose origin lookup retries over the production backoff with `time.Sleep`, 1, 2 and 4 s (internal/fabricengine/pushclassified.go, `awaitOrigin`; `RunPush` carries no sleep). PATTERN-test-speed forbids waiting out a production duration in any tier."
    fix: "In card 9, which already edits `pairteardown#RunPush` and `#PushRun`, add a `RunPush.BackoffSleep` field that `PushRun` copies into the push's `SyncOptions.BackoffSleep`, nil in production. In card 17, give `catchUpPushSeams` the same sleep, put it on its `RunPush`, have the test pass one that returns at once, and restate the test's seconds."
prior:
  - id: F1
    status: resolved
  - id: F2
    status: resolved
  - id: F3
    status: resolved
  - id: F4
    status: resolved
  - id: F5
    status: resolved
  - id: F6
    status: resolved
  - id: F7
    status: resolved
  - id: F8
    status: resolved
  - id: F9
    status: resolved
  - id: F10
    status: resolved
---

### [BLOCKING] Board writes are never committed or pushed

**Location:** `_lym/plan/06-pusher-target-kinds.md#Intent`

**Issue:** Round 2 made card 7's board spawn a line the verb accepts, and it replaces `lym board sync`.
The pusher it starts asks only whether commits are unpushed.
A board write leaves none: it saves `board.json` and the rendered files, and the sync was what committed them.
`syncBoardIfOwed` in `internal/tendercli/deps.go` checks uncommitted changes today for that reason.
No planned test runs the pusher over a board write left uncommitted.

### [BLOCKING] A non-squash landing has no commit to carry the trailer

**Location:** `_lym/plan/10-parent-push-git.md#Intent`

**Issue:** Every read that rebuilds a landing from git depends on a commit with the trailer.
A fast-forward discards the landing message, and `concludeMergeSides` commits nothing for it.
Both ways to reach it are supported settings, and no planned test runs one.

### [BLOCKING] The `done` write is judged on a Bolt not yet pulled

**Location:** `_lym/plan/11-parent-pusher.md#Intent`

**Issue:** The pull sits in step 3, after both steps that act on an owner.
The dobby tick's settle path never reaches it.
A run adopted elsewhere whose tip already reached origin can get a `done` write from this machine.

### [BLOCKING] A followed way forward leaves the prime's parent refused

**Location:** `_lym/plan/11-parent-pusher.md#Intent`

**Issue:** After the rebase nothing on this machine matches any clearing rule.
Later local landings record entries that are never pushed,
and pull-request landings halt on the stale refusal (card 13).
The planned rows cover the case with a newer landing only, which settles and so clears.

### [BLOCKING] Batten's own network waits are invisible

**Location:** `_lym/plan/20-dobby-push-summary.md#Intent`

**Issue:** Card 2 edits `loomcli`'s driver wiring only.
Card 16 turns Land's unreachable pull into a stop that batten's driver is meant to wait out.
Round 2 noted this under Rejected as outside its changed hunks;
the `Waits` read that round added claims full coverage and does not have it.

### [BLOCKING] The catch-up test sleeps through the push backoff

**Location:** `_lym/plan/17-catch-up-hands-to-pusher.md#Intent`

**Issue:** Round 2 shortened the lock wait.
The row with the prime's origin repointed to a missing path still takes the production retry schedule,
and the card's estimate of about six seconds is mostly that sleep.

## Focus verdicts

- Records hold on every push path: branch pushes stop short, a removal's included.
  A tag residual is listed under Rejected.
- Lost or never-written ledger: holds for a squash landing, through the re-record and the held-records ref; F2 otherwise.
- Dropped landing: remembered through `Dropped` and reset for a squash commit; F4 is the way it stays stuck.
- Pair entry identity: every named writer sets `Worktree` and `Slug`, and the drop row starts from an existing entry.
  Not a defect.
- Beam push before a ledger entry: the card 10 ceiling covers it for a squash landing; F2 otherwise.
- Pushes off or git off: card 19 removes the pair before and after pushes return.
  The board target's skip handling is part of F1.
- Push interval: only the landing, settle and teardown starts set the mark.
  During an outage the entry stands and the interval holds.
  After a landed push the entry goes and the next commit pushes at once; D2 limits "while the target is owed"
  and D3 removes the entry, so this is not raised, though card 7's sentence on spawned commits overstates it.
- Board pusher command line: accepted by the verb as card 6 states it, and tested; F1 is what it then does.
- Push summary coverage: beam members are covered; batten's own run is not (F5).
- Test waits: F6. The stated tier0 estimates stay within the budget.

## Rejected

Not raised as findings: a new finding in this incremental round must be `BLOCKING` and sit on a hunk round 2 changed.

- Outside the changed hunks, serious enough for the parent to read:
  card 14 evaluates ignore rules in one fixed scratch repository, `<HubScratchDir>/<IgnoreCheckDirName>`.
  Finalize's merge-in runs without the parent lock, so two runs can overwrite each other's ignore files
  and delete files their own base does not ignore.
  A directory per call, removed after use, settles it.
- Outside the changed hunks, serious enough for the parent to read:
  card 12's `TestLandingDeps_LocalLanding` expects done with origin unreachable.
  `syncParent`'s fetch then fails, and D5 and card 12's own intent require a `git-transport` stop.
  The row should make only the push fail.
- Below `BLOCKING`: `removePair` tags the whole records tip locally (`tagWeftTipForArchive`) before card 9's cap applies.
  A pending-teardown re-run, a later teardown of the slug or the printed hand push can send that tag, held commit included.
  D13 forbids branches only, but card 9 says no tag reaches it.
- Below `BLOCKING`: no card runs a pair's way forward, `lym fabric push`, and sees the next tick drop the entry
  and post the all-clear, which D3's acceptance criterion asks.
- Below `BLOCKING`: card 12 removes the last call of `parentMerger.PushBranch` and keeps the method.
- Outside the changed hunks, reachability not confirmed: `HandOverTips` pushes a beam pair's owed code tip uncapped,
  which the round 2 fixer-report deferred to this round.

## Proposed-fix check

`lym loom validate-plan --overlay` reported no blocking finding on the `{}` baseline.
The added targets and Uses lines of F1, F2 and F5, overlaid together, reported none either.
F6's fix adds no target.
A first F6 shape that added the `RunPush` targets to card 17 was refused by `uses-later-target`, so the field goes in card 9.
The other fixes change no targets.
