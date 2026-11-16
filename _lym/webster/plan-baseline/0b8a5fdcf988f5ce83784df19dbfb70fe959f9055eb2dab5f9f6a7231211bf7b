---
format: 5
approved: true
language: go
---

# Plan: a transition pushes its records before the run acts, and a takeover entry hands the run over

A shed transition commits its records and pushes them synchronously, records side first and code side after, before the next producer starts.
The tracked intents, the pull-request record and the landing's board `done` push the same way, and a push owed on an unreachable origin or a held teardown lock holds the run at its transition and shows in `lym beam status`.
A rejected records push reads the run's own record on origin first: another machine's takeover entry hands the run over (processes ended, local tips archived, a hand-over mark written, the local pair removed) and nothing is replayed.
A code tip the run's last transition names but origin lacks makes the run wait, and the owner gives it up after a 30-minute grace.

## Card Index

1 — runfsm-push-owed — rename PushOwedOffline to PushOwed, carry its last error and the missing code tip, add the code-tip grace
2 — takeover-entry — the takeover entry in the status record, the hand-over directory name and the takeover test kit
3 — pair-push-records-first — PushPairAnchored takes PairPush, pushes records first and code only after, and owes a push past the teardown wait
4 — detached-records-first — coalescePushSides pushes records first; the pending-flag and failed-remove respawns retire
5 — rejected-push-takeover-read — a rejected records push reads origin's takeover entry before any replay
6 — run-hand-over — fabric's hand-over git steps and pairteardown's PushRun and HandOver; commit-records hands over
7 — read-run-owed-and-tip — ReadRun fills PushOwed from failed-push records and the code-tip wait from origin; the owner's tip guard
8 — transition-sync-push — statuscommit pushes synchronously under a gate admission and retries an owed push
9 — intent-sync-push — tender pause and resume, batten cancel and the pull-request record push synchronously
10 — board-done-pushed — a synchronous board status write for the landing's done
11 — finalize-push-order — Finalize pushes the parent, then its records, then the board done, reading the owner before each
12 — code-tip-grace — the owner's dobby gives up a missing code tip after the grace and fast-forwards a landed one
13 — status-shows-owed — lym beam status shows the owed push and the missing code tip
14 — pattern-run-fsm — PATTERN-run-fsm added, PATTERN-pair-remote changed, their backgrounds

## Shared Decisions

### Decision: code-tip-is-warp-sha

- **Decision:** the records commit of a transition names its code tip through the `Warp-SHA` trailer, the code HEAD it was made at, and carries a second trailer `Run-Transition: <run id>` (`fabricengine.TransitionTrailerKey`, card 6).
  Both transition seams commit through fabric's trailer commit (`CommitAnchoredPathsWithTrailers`, card 8), which writes `Warp-SHA` and the `Run-Transition` trailer they pass;
  card 12's give-up commit writes both itself, its `Warp-SHA` being the previous transition's.
  "The tip the run's last transition on origin names" is the `Warp-SHA` of the newest commit on origin's fetched records branch of the slug whose `Run-Transition` names the slug.
- **Rationale:** D1 asks the commit to name its code tip, and `Warp-SHA` already names it.
  Fabric's other weft commits carry `Warp-SHA` too (`commitWeftLocked`'s commits, made by `lym fabric commit` and `lym fabric sync`, and the anchor re-anchor commit), so the newest `Warp-SHA` commit need not be a transition;
  `Run-Transition`, written only by the transition seams and the give-up, marks which commits are.
- **Applies to:** cards 6, 7, 8, 12, 14

### Decision: hand-over-split

- **Decision:** `internal/fabricengine` detects the takeover (`*TakeoverError`, `ErrHandedOver`) and does the git steps (`HandOverTips`, `ArchiveParentMerge`); `internal/pairteardown` runs the hand-over (`HandOver`) and the run's push that triggers it (`PushRun`).
  Every synchronous run push outside fabric goes through `PushRun`.
- **Rationale:** fabric owns git on the pair and reads no board (`PATTERN-fabric-boundary`); pairteardown owns process ends and pair removal and already imports fabricengine.
- **Applies to:** cards 5, 6, 8, 9, 11, 12

### Decision: names-from-the-caller

- **Decision:** `fabricengine.MachineNames` carries the names that read as this machine; `pairteardown.ThisMachine` fills it from `machinename.Names`, and the registry part adds its names there.
  A zero value turns the takeover read off, and only `CoalescePushBothAt`, a fixture's push, passes one.
- **Rationale:** D2 has the caller hand fabric the name set; the exported `PairPush` struct lets later parts add fields without re-signing `PushPairAnchored`.
- **Applies to:** cards 3, 5, 6

### Decision: owed-is-a-failed-push-record

- **Decision:** an owed push is a failed-push record of class `transport`, `deferred` or the new `teardown_lock`; `ReadRun` reads it into `PushOwed` with its error and worktree, and `fabricengine.IsPushOwed` tells a seam to retry.
  The record carries the runs it serves (`FailedPush.Runs`, unioned across failures), so an owed push of the prime's records branch holds only the runs whose pushes failed it.
  The refusal's way forward is `lym fabric push` in the owed worktree, which lands the side and clears the record, then the verb's re-run.
  Any other failure halts the run at the transition with its record and notice.
- **Rationale:** the record and its creel notice already exist per side and branch and are cleared by a landing push, so the owed state survives a restart with no new file.
- **Applies to:** cards 3, 7, 8, 9, 13

### Decision: out-of-part

- **Decision:** the takeover entry's production writer and the drop of hand-over marks on an adopt naming this machine come with adopt (D6); the hold (D5), the registry (D3) and the catch-up (D4) are other parts.
  The run claim and the owner already push synchronously through `SetOwners`, so card 10 adds only the landing's `done`.
  `internal/pauserecord`, `internal/pullrequest` and `internal/tenderengine` need no change: `pauserecord.Admit` serves as is, the pull-request record's push is its committer's (card 9), and tender's pause already holds the run's processes through its pause entry before the intent's push.
  Pawl's `SpawnPush` (`internal/pawlengine/beamup.go:55`, "spawn the beam's push") pushes the beam pair a beam-up just made, as `add`'s spawn does, and carries no run transition, so it stays detached.
  `Teardown.removeCancelledRunRecord`'s `SpawnStatusPush` (`internal/pairteardown/followup.go:107`) removes the record of a run whose cancel finished, so it carries no transition a run acts on, and it stays detached.
  Tender's hub-resume spawns, `tendercli.HubDeps`' `PushPair` and `PushPrimeRecords` (`internal/tenderengine/resume.go:247-254`), push what was committed before the pause once it lifts and write no transition of their own, so they stay detached.
  An owed intent push of the prime's records branch lands with that branch's next push: any run's transition push, `lym fabric push` from the prime, or D4's catch-up at the next dobby start (the arbiter's reading of D9's bound).
- **Rationale:** each function added has a production caller in this part (`PATTERN-test-only-code`).
- **Applies to:** all cards

## PATTERN changes

- `PATTERN-run-fsm` — adds: Reading or acting on a run's lifecycle state: only `internal/runfsm`'s states and act rule; a transition commits and pushes before the run acts; a run whose record holds another machine's takeover hands over, never replays.
- `PATTERN-pair-remote` — changes: Pushing run records or tearing down a pair: the code branch goes along after the records side, both before the run acts; a pair's remote steps hold the slug's teardown lock; teardown only via `internal/pairteardown`.
