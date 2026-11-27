---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: "_lym/plan/09-records-hold.md#Intent"
    summary: "Card 9 says every push of a task pair meets the hold, but a non-local removal pushes the local records tip to the pair's records branch on origin through `fabricengine#archiveWeftTip` (internal/fabricengine/archive.go:138, `pushBranch` set for a removal). Card 19 lets `lym fabric remove` remove a dropped-landing pair, so the held records commit reaches an origin branch, against D13 and D5."
    fix: "In card 9, add `internal/fabricengine#Topology.removePair`, `internal/fabricengine#remoteTeardownJob`, `internal/fabricengine#runRemoteTeardown` and `internal/fabricengine#archiveWeftTip` to Edit. `removePair` reads the slug's hold through `recordsPushCeiling` before its first mutation and carries it on the job, since the pair's ledger entry goes as soon as `Remove` returns and the child is detached. `archiveWeftTip` then pushes the records branch only up to `<hold>^`, and the archive tag still names the whole tip. In card 19, make the dropped-landing row of `TestCommand_RemoveRefusesWhileALandingStands` assert that origin's records branch lacks the held commit after the removal, with and without `--remote`."
  - id: F2
    severity: BLOCKING
    class: design
    location: "_lym/plan/09-records-hold.md#Intent"
    summary: "A beam pair's code ceiling is derived only from ledger landings, so a landing commit with no entry yet (between the parent-side merge and `RecordLanding`, after a kill there, or after a lost ledger) is pushed whole by the beam's ordinary pusher pass. No owner is read, against PATTERN-owner-machine, D5 and card 9's own claim. Once origin holds it, no pass re-records it and the run never gets `done`."
    fix: "In card 10, add `internal/pairteardown#PushRun` and `internal/beamcli#hubLandDeps` to Edit and `internal/beamcli/doc.go` to Uses. With `RunPush.CodeCeiling` empty on a beam pair, the ceiling is the first parent of the oldest first-parent commit past origin that is either an owed ledger tip or a `fabricengine.LandingCommits` commit, and `PushBeam` refuses on either. In card 11, have `pusherDeps` wire the parent seams for every beam pair target, one with no landing included, and drop the clause that a pair with no landing runs no parent step. Add a tier1 row: a beam pair with a `Lym-Landing` commit past origin and no ledger entry keeps that commit off origin through `PushRun`."
  - id: F3
    severity: BLOCKING
    class: design
    location: "_lym/plan/19-teardown-hands-to-dobby.md#Intent"
    summary: "`LandingStands` counts an unrecorded `Lym-Landing` commit past origin whenever git is on. With pushes off, Finalize writes the trailer commit and `done` but no landing entry (card 12), so `Teardown.RemovePair` refuses Worktree-Teardown and `lym fabric remove`, against D13 and D11 (with git or pushes off the pair is removed as today)."
    fix: "In card 19, have the trailer clause of `LandingStands` apply only with both git and pushes on, and skip a run whose board entry is `done`. In card 11, apply the same `done` skip to the landing target's widened `Ahead` and to `settleParent`'s entry condition, so a commit left by a pushes-off landing starts no endless pusher. Add a tier1 row: after a local landing with pushes off and git on, `Teardown.RemovePair` removes the pair, before and after pushes are turned on again."
  - id: F4
    severity: BLOCKING
    class: design
    location: "_lym/plan/11-parent-pusher.md#Intent"
    summary: "A drop is remembered only as the pair entry's `HeldRecords`, which `RemovePair` deletes while the dropped commit can still stand on the local parent. Card 11 says the hand-over of a run owned elsewhere resets that parent, but `HandOverOwnedElsewhere` passes no `Parent` (internal/pairteardown/handover.go:220), and `parentAheadByOwnMergeOnly` rejects a one-parent squash commit. The commit is then re-recorded and landed twice, or leaves the parent diverged so later landings are Stuck."
    fix: "In card 11 step 2, reset the local parent through `ResetToOrigin` for a dropped run owned elsewhere too, before the hand-over, when the landing commit alone puts it ahead. Record the dropped run on the parent target's entry (a card 4 field), kept while `LandingCommits` still lists its commit, and have the re-record, step 4 and card 19's `LandingStands` read it, so a removed pair never turns a dropped commit back into a landing. A push carrying a still-listed dropped commit stays refused with step 2's rebase way forward. Add `TestParentPush` rows: an owned-elsewhere drop leaves the local parent at origin's, and a dropped run owned here with a newer landing on it, then removed, is neither re-recorded nor pushed."
  - id: F5
    severity: BLOCKING
    class: design
    carries: F6
    location: "_lym/plan/11-parent-pusher.md#Intent"
    summary: "The rebuild from git is incomplete. A re-record happens only past Finalize, so the pair's records tip it holds is later than Finalize's records commit, and the push up to `<hold>^` sends that commit while the parent push is owed. A dropped landing's `HeldRecords` lives only under `.lym`; after the reset removes the trailer commit, a lost ledger releases the hold (PATTERN-lym-files, D5, D13)."
    fix: "In card 11, have a re-recorded landing hold the oldest commit of the pair's records branch past its tracking ref, not the tip, so nothing card 10 kept back goes out before the settle. Have the drop also keep the held commit where git holds it, such as a local ref in the records store that card 9's `recordsPushCeiling` reads and the pair's removal deletes. Add `TestParentPush` rows: with the ledger deleted after a landing and the done transition committed, origin still lacks Finalize's records commit after the re-record; with the ledger deleted after a drop, a pair push still stops short of the held commit."
  - id: F6
    severity: BLOCKING
    class: design
    carries: F3
    location: "_lym/plan/11-parent-pusher.md#Intent"
    summary: "No card has the ordinary writers of a pair's entry (card 5's pass, card 6's starter and `--push-now`, card 19's mark) set `Worktree` or `Slug`, and card 11's drop sets them only when it creates the entry. A drop onto an existing entry leaves `HeldRecordsCommit` unable to find the hold by slug, and `PruneLedger` and the summary read an empty `Worktree`."
    fix: "In cards 5, 6 and 19, state that every write of a records target's entry sets `Worktree` to the target's code worktree and `Slug` to the target's slug. In card 11 step 2, have the drop set both on an existing entry as well as a new one. Add a `TestStart` assertion on the written entry's `Worktree` and `Slug`, and make `TestParentPush`'s drop row start from a pair entry that already exists."
  - id: F7
    severity: BLOCKING
    class: design
    location: "_lym/plan/07-spawns-start-the-pusher.md#Intent"
    summary: "Every commit and board spawn passes `--push-now`, whose mark skips the interval wait. Each commit then pushes at once, during an outage too, against D2 (at most one push per interval in force while owed), which gives the mark only to the landing, settle and teardown paths (D5, D13)."
    fix: "In card 7, have `pushSpawnCommand` and `SpawnSync` leave `PushNow` unset, so a spawn starts a pusher and a running one picks the commit up at its next pass; keep the mark for card 11's `RecordLanding` and `WakePair` and card 19's teardown push. Change card 7's test lines, and the overview's decision `a spawn starts the pusher through its command line`, to the line without `--push-now`. Add a card 5 row: a second `Run` started by a spawn within the interval waits out its rest."
  - id: F8
    severity: BLOCKING
    class: consistency
    location: "_lym/plan/07-spawns-start-the-pusher.md#Intent"
    summary: "Card 7's board spawn and its test pin `lym loom pusher --board` with no `--worktree`, but the pusher verb marks `--worktree` required (internal/loomcli/pusher.go) and no card changes that or says how `--board` finds the hub. As written the detached child fails unseen and the board is never pushed (D1)."
    fix: "In card 7, have `SpawnSync` pass `--worktree` with the board directory it is given. In card 6, state that with `--board` the pusher uses `--worktree` only to locate the hub and builds `BoardTarget` over the prime. Have card 7 pin that line, and add to card 6's `TestPusherCmd_PushNowWakesAndExitsAtAHeldLock` a row that runs exactly the arguments `SpawnSync` builds."
  - id: F9
    severity: BLOCKING
    class: design
    location: "_lym/plan/20-dobby-push-summary.md#Intent"
    summary: "The summary takes waits from the prime dobby's listed runs, which are only the runs whose parent is the prime (internal/battencli/dobbyreads.go:101). A Go driver waiting in a beam member's run mails nothing (card 3) and appears in no summary, against D3 (every run the driver waits on) and D6."
    fix: "In card 20, give `PushSummaryDeps` a `Waits` read over every run of the hub, filled in `hubDobbyDepsStarting` from `battencli.HubRunSlugs` and `sheddriver.ReadDriveWait` over each pair's record and driver lock, and start the owed period from those waits' `Since`. Add a `TestPushSummary` row for a wait of a run the prime's dobby does not list."
  - id: F10
    severity: BLOCKING
    class: design
    location: "_lym/plan/17-catch-up-hands-to-pusher.md#Intent"
    summary: "`TestPushOwedForCatchUp` holds the pair's push lock while the real push runs with `fabriclayout.StatusPushLockWait`, 30 s, so the test waits out a production duration, which PATTERN-test-speed forbids in any tier; the card's estimate of about six seconds cannot hold."
    fix: "In card 17, give `catchUpPushSeams` a push-lock wait field that `HubCatchUp` fills with `fabriclayout.StatusPushLockWait` and that `pushOwedForCatchUp` puts on its `RunPush`; the test sets it to a few milliseconds. Restate the test's seconds."
prior:
  - id: F1
    status: resolved
  - id: F2
    status: resolved
  - id: F3
    status: open
  - id: F4
    status: resolved
  - id: F5
    status: resolved
  - id: F6
    status: open
  - id: F7
    status: resolved
  - id: F8
    status: resolved
  - id: F9
    status: resolved
  - id: F10
    status: resolved
  - id: F11
    status: resolved
  - id: F12
    status: resolved
  - id: F13
    status: resolved
  - id: F14
    status: resolved
  - id: F15
    status: resolved
  - id: F16
    status: resolved
  - id: F17
    status: resolved
  - id: F18
    status: resolved
  - id: F19
    status: resolved
---

### [BLOCKING] Removing a pair pushes the held records commit

**Location:** `_lym/plan/09-records-hold.md#Intent`

**Issue:** Card 9 hooks the hold into `PushPairAnchored` and `coalescePushSides` only.
A removal's teardown child calls `archiveWeftTip` with `pushBranch` set,
which pushes the local records tip to the records branch with `PushCommit`.
Card 19 says a later `lym fabric remove` of a dropped-landing pair passes its check while its pushes stop short of the held commit.
They do not.
Without `--remote` the branch stays on origin holding it.

### [BLOCKING] A beam's pusher can send a landing before its owners are read

**Location:** `_lym/plan/09-records-hold.md#Intent`

**Issue:** `PushRun` derives its code ceiling from `pushledger.OwedTips` alone.
`HoldPairPushLock` (card 12) locks the landing run's pair, not the beam parent,
so a running beam pusher can push the merge commit before `RecordLanding` writes the entry.
A kill between the two, or a deleted ledger, leaves the same state for good.
Card 11 also contradicts itself here:
it wires parent seams for every pair target, yet says a pair with no landing runs no parent step.

### [BLOCKING] Pushes off, the landed pair can no longer be removed

**Location:** `_lym/plan/19-teardown-hands-to-dobby.md#Intent`

**Issue:** `landingTexts` adds the trailer on every local landing, and with `PushSkipped` no entry is written.
The commit stays past origin's tracking ref, the pair entry holds no `HeldRecords`,
and `LandingStands` asks only whether git is on.
`RemovePair` therefore refuses.
The planned pushes-off test drives only `teardownPushSeams`, and the `TestTeardown_CallSequence` rows run with git off,
so no test reaches this.

### [BLOCKING] A dropped landing is forgotten while its commit still stands

**Location:** `_lym/plan/11-parent-pusher.md#Intent`

**Issue:** `HandOverOwnedElsewhere` calls `HandOver` with an empty `RunPush`, so `HandOverTips` gets no parent to reset.
Even with one, `parentAheadByOwnMergeOnly` accepts an extra commit only when it is a merge, and `landing.squash` defaults to true.
`RemovePair` then deletes the pair entry with its `HeldRecords`.
The re-record reads the pair as gone, hence past Finalize:
with the board not yet `done` it re-records and pushes the dropped landing,
and with `done` it stays on the parent, which is diverged from origin.
`lym fabric remove` of a dropped run owned here, with a newer landing on its commit, loses the memory the same way.

### [BLOCKING] A lost ledger still releases records the landing holds (carries F6)

**Location:** `_lym/plan/11-parent-pusher.md#Intent`

**Issue:** The re-record holds the records tip at the re-record.
It runs only when the run is done or its pair is gone, so the done transition has been committed after Finalize's commit,
and `<hold>^` includes Finalize's commit.
The round 1 fix text asked for the tip; that choice is wrong.
Card 10's trailer read cannot cover a dropped landing whose commit was reset off the parent.

### [BLOCKING] Pair entries are written without `Slug` or `Worktree` (carries F3)

**Location:** `_lym/plan/11-parent-pusher.md#Intent`

**Issue:** The round 1 fix named the writers for the parent keys and for a pair entry the drop creates.
A pair that was ever owed already has an entry, written by card 5 or 6 with neither field named.
The drop then adds `HeldRecords` to an entry `HeldRecordsCommit` cannot match by slug,
and the next pair push sends the held commit.

### [BLOCKING] Every commit skips the push interval

**Location:** `_lym/plan/07-spawns-start-the-pusher.md#Intent`

**Issue:** The spawned verb sets the entry's `PushNow` before it takes the lock (card 6),
and card 5 skips the wait for a marked entry.
Status transitions and board writes spawn often, so the interval and its backoff rarely apply.
D2 and its rationale, fewer pushes and less traffic, are not met.

### [BLOCKING] The board spawn's command line is one the verb refuses

**Location:** `_lym/plan/07-spawns-start-the-pusher.md#Intent`

**Issue:** `pusherCmd` ends in `MarkFlagRequired(runpusher.WorktreeFlag)` and resolves its target from that flag, never the cwd.
`SpawnSync` gets only the board directory, and card 7 pins `--board --push-now` with nothing else.
`TestSpawnSync_RegistersUnderTheHubOwner` checks the line, not that the verb accepts it.

### [BLOCKING] Waits under a beam reach no summary

**Location:** `_lym/plan/20-dobby-push-summary.md#Intent`

**Issue:** Only the prime's dobby posts the summary, and `DobbyRuns` lists the runs whose parent is that dobby's worktree.
A beam's dobby has a nil `Ledger`.
D6 names the summary as the one place a long wait shows.

### [BLOCKING] The catch-up test waits 30 seconds

**Location:** `_lym/plan/17-catch-up-hands-to-pusher.md#Intent`

**Issue:** The card fills the test's seams with `HubCatchUp`'s real ones, including today's `RunPush`.
Its `LockWait` is `fabriclayout.StatusPushLockWait`, and the test holds that lock for the whole push.

## Rejected

Not raised as findings: this round is incremental, so a new finding must be `BLOCKING` and sit on a changed hunk.

- Outside the changed hunks, and serious enough for the parent to read:
  no card wires batten's own driver (`battencli#driverDepsFor`, internal/battencli/drive.go) with `NetworkWaitCeiling`
  or `WaitRecordPath`, and card 2 edits only `loomcli`'s.
  A Land `git-transport` stop (card 16) is then re-stepped every 15 s with no wait record,
  so it never enters the summary, against D6 and D3.
- Outside the changed hunks: card 10 needs its refusal to name the landing commits,
  but only card 18, which runs later, has `FailedPushWayForward` pass commits through.
  Card 18's `codeDivergedWayForward` also changes parameters with no re-sign arrow.
- Below `BLOCKING`: cards 5, 9 and 10 need `pushclassified.go` changes (single attempt, push to a ceiling)
  that no card targets, and card 19's `endRunPhase` read needs a `cancelDeps` field no card targets.
- Below `BLOCKING`: card 12's resume records the parent's head, not the run's trailer commit,
  so a hand commit made on the prime after a killed Finalize would be pushed.
- Below `BLOCKING`: card 11's push of an older tip and its settle clear the `held_back` record just written,
  so a second notice goes out.
- Below `BLOCKING`: card 20 names a refused target with its way forward, but `TargetRefused` returns only a bool.
- Below `BLOCKING`: card 11's `lym loom resume` way forward is refused for a `running` run with no live driver,
  though its other way forward, the rebase, works.
- Below `BLOCKING`: the tier0 rows card 19 adds to `TestEndForCancel` and in
  `TestWorktreeTeardown_AFalseLevelEndsDoneWithThePairStanding`, and card 11's `TestStart` row, state no seconds.

## Proposed-fix check

The added targets of F1 and F2, with `internal/beamcli/doc.go` under card 10's Uses, were overlaid together through
`lym loom validate-plan --overlay`, which reported no blocking finding; the `{}` baseline reported none either.
The other fixes add, drop and move no targets.
