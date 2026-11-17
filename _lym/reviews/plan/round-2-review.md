---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: _lym/plan/15-tender-bolt-hold.md:43
    summary: "Card 15 does not order its hold refusal before the pause-intent clear, so a refused `lym tender resume <slug>` can still drop the run's pause intent"
    fix: >-
      In card 15's Intent, state that `tenderengine.Resume` judges the hold rule for a slug only a `bolt-hold` entry covers
      before it calls `req.Clear`, so a refused resume clears no pause intent and pushes nothing.
      Add one `TestResume` row to the existing resume-rules bullet:
      a refused resume of a `bolt-hold`-only slug whose run carries a pause intent leaves `Clear` uncalled and the intent standing.
      The fix changes no target and no verify line, so it needs no `validate-plan --overlay` run.
  - id: F2
    severity: MEDIUM
    class: decision
    location: _lym/plan/23-retire-owner-pause-callers.md:91
    summary: "Card 23 removes orchCLI.pauseOwned but names no replacement seam, no source for the beam flag that sets endDobby, and no ruling on the owner_pauses report key and the paused wording"
    fix: >-
      In card 23's bring-up paragraph, name the replacement seam and its filler.
      Use `endOwned func(location *lymcwd.Location, slug string, endDobby bool) error` on `orchCLI`, filled in `wireCLI` by
      `pairteardown.EndSubjectProcesses(prime, slug, endDobby)`, with a nil seam skipped and warned as `pauseOwned` is today.
      Name where `endDobby` comes from:
      an `isBeam func(slug string) (bool, error)` seam filled in `wireCLI` from `boardengine.ReadOwners(...).Entries[slug].Beam`,
      because `fabricengine.EntryOutcome` carries no beam flag.
      Say that the report key `owner_pauses`, `orchcli.startReport.OwnerPauses` and `beamcli`'s `startEntry.OwnerPauses` keep their names and shape
      (the slug and owner of each entry whose processes ended), and give the two warning texts in `pauseOwnedRuns` in ended wording
      ("was not ended" and "could not be ended"; "re-run `lym beam start`, which ends it again").
      `TestStart_ReconcilesTheHubFirst` and `startHarness.runWith`, both already listed, assert the new seam's calls with the flag
      and the two reworded warnings; `startHarness`'s four test-local fields keep their names so the card adds no target.
      The fix changes no target and no verify line, so it needs no `validate-plan --overlay` run.
  - id: F3
    severity: MEDIUM
    class: design
    location: _lym/plan/21-dobby-catch-up.md:50
    summary: "Card 21's hand_over trigger keys on a path-exists PairPresent while the hand-over removes only a registered worktree, so a leftover pair directory owes the catch-up on every tick"
    fix: >-
      In card 21's list of owing triggers, bound the fourth trigger.
      `RunView.PairPresent` is `markerExists(pair.WorktreePath())` in `battencli.DobbyRuns`, a path check,
      while `pairteardown.HandOver` removes the pair only when `Teardown.TaskWorktreeGone` reads it as a registered linked worktree;
      for a directory left standing with no registration, `HandOver` removes nothing, the catch-up succeeds and resets its delay,
      and the next tick lists the run again as `hand_over` with `PairPresent` true.
      State that a run the catch-up has handed over owes no second catch-up from this trigger while it still lists as `hand_over`,
      held in per-run daemon memory that is cleared when the run stops answering `hand_over` or is no longer listed,
      and that card 20's `DobbyPairSlugs` reads "whose pair stands here" by the same predicate as `PairPresent`.
      Add a `TestRun_CatchUp` row:
      a listed `hand_over` run whose pair stays present after a successful catch-up owes the catch-up once, not on each tick.
      `internal/dobbyengine#daemon` is already under card 21's `**Edit:**`, so the fix adds no target and needs no `validate-plan --overlay` run.
---

### [MEDIUM] Card 15 does not order its hold refusal before the pause-intent clear

**Location:** `_lym/plan/15-tender-bolt-hold.md:43`, `internal/tenderengine/resume.go` `Resume`.

**Issue:** The focus asked whether `lym tender resume <slug>` under a standing hold reaches card 15's hold rules.
It does.
`runfsm.Aside` already has `Hold`, `Judge` drops `Inputs.Held` under it, and card 15 adds `Hold` to `tendercli`'s `resumeAside`,
so card 13's held refusal no longer stops the verb before `Resume`'s own rules.
The gap is the order inside `Resume`.
Today `Resume` judges the verdict, then calls `req.Clear` (which clears the run's pause intent in its batten record and pushes it), then runs `liftEntries`;
the existing "has no pause of its own" error sits inside the `liftEntries` callback, after the clear.
Card 15 says the refusal for a slug only a `bolt-hold` entry covers happens "with nothing lifted", and does not say it precedes the clear.
An implementation that copies the existing pattern refuses after `Clear` has dropped and pushed the intent.
Card 22 turns each standing adopt entry into a pause intent and tells the operator to run `lym tender resume <slug>` only once the check is green.
An early resume would then be refused while the intent is gone, and the run would go on unasked when the hold clears,
which breaks D5's "so no run resumes unasked".
The Intent states no bound on what a refused resume can still change, and no planned test reaches that.

### [MEDIUM] Card 23 leaves the bring-up's replacement seam and its inputs open

**Location:** `_lym/plan/23-retire-owner-pause-callers.md:91`, `internal/orchcli/cli.go:115`, `internal/orchcli/start.go:419`.

**Issue:** The card deletes the `pauseOwned` field and says `pauseOwnedRuns` ends each entry's processes through `pairteardown.EndSubjectProcesses(prime, slug, endDobby)`.
Its test text says the processes end "through the seam that replaces the pause", but no seam is named, so the implementer and the tests must invent its name and signature.
`endDobby` is "set when the entry is a beam", but `fabricengine.EntryOutcome` has no beam flag, so the card does not say where the flag is read.
`pauseOwnedRuns` also words its warnings as "was not paused" and "re-run `lym beam start`, which pauses it again",
and `owner_pauses` is a report key shared by `orchcli.startReport` and `beamcli`'s `startEntry`; the card rules on neither.
Both are open choices that the card's own tests assert (`start_test.go` asserts the key and the warning text).
Whether `EndSubjectProcesses` with a task slug ends that task's own processes was confirmed in `runproc.EndSubject`, which ends the subject's own processes for any slug,
so the call itself is sound; only its wiring is unspecified.

### [MEDIUM] Card 21's hand_over trigger can re-owe the catch-up on every tick

**Location:** `_lym/plan/21-dobby-catch-up.md:50`, `internal/battencli/dobbyreads.go:218`, `internal/pairteardown/handover.go:91`.

**Issue:** The focus asked whether a listed `hand_over` run with no local pair, or one that never had a pair, can owe the catch-up on every tick.
It cannot.
`DobbyRuns` lists every batten record of the worktree, and the card gates the trigger on `RunView.PairPresent`, with a `TestRun_CatchUp` row for the false case.
A pair that stands is removed by the hand-over, after which `PairPresent` reads false.
The remaining loop is a pair directory that exists on disk but is not a registered linked worktree.
`PairPresent` is `markerExists(pair.WorktreePath())`, an `os.Stat`, and the hand-over's removal is conditional on `TaskWorktreeGone`, which reads the registration.
For such a directory `HandOver` reaches `pair == nil`, removes nothing and returns `ErrHandedOver`, which card 17 maps to nil,
so the catch-up succeeds, clears its wait and resets its delay.
The next tick's `drivePasses` lists the run as `hand_over` with `PairPresent` true and owes again, with a Bolt pull and pair fetches on each try and no backoff, since each try succeeds.
Card 20 also leaves "whose pair stands here" in `DobbyPairSlugs` without a predicate, so the trigger's set and the catch-up's scope may differ.

## Rejected

- Focus 2, card 13 with card 15 (`resumeAside`): reaches card 15's rules, not a defect beyond F1's ordering.
  `runfsm.Aside.Hold` exists (`internal/runfsm/act.go`), `Judge` clears `Held` under it, and card 15 adds `Hold` to `resumeAside`;
  card 13's impact line names that handover.
- Focus 3, test-seam fields reshaped or removed (cards 8 and 23): not a defect.
  Every test member that assigns `Deps.Owner`, `StartDeps.PrimeOwner`, `Deps.PauseOwned`, `Deps.LiftOwnerPause`, `Deps.Machine`, `InnerRunDeps.PauseOwned`,
  `prePushCheck.pauseOwned` or `orchCLI.pauseOwned` is under the card's `**Edit:**`:
  `newLoopHarness`, `newPassHarness`, `fakeStart.baseDeps`, `runAnAdoptHold`, the five loop and handover tests card 8 names, `TestRun_ForeignOwner`,
  `newLockHarness`, `TestPrePushCheckGuard`, `stoppingPrePushCheck`, `startHarness.runWith`, `TestStart_ReconcilesTheHubFirst` and `TestHubDobbyDeps_ABeamWithNoOwnerOfItsOwnIsUnderARootAdopt`.
  The test-local fields `lockHarness.ownerPauses` and `heldByOwner` and `startHarness.pauseOwned` go unread but compile, so no listing is forced.
- Focus 4, card 16 with card 13 (batten start): not a defect.
  Card 16 adds no refusal to `refuseStartOwnedElsewhere`; `startFirstPass` runs `refuseByAct` first and card 13's `Held` makes it refuse, which `TestAct` and `TestReadRun` reach.
- Focus 5, card 17's route into `HandOver`: not a defect.
  The Intent's Bound states what the forced local removal lets through and what bounds it:
  it runs after `HandOver` returns the archive's error, pushes only commits origin lacks, never rewinds or force-pushes, and writes no board.
  The empty `Subject` and `At` of the synthetic `Takeover` are harmless, since `takeoverStop` compares only `Machine`.
- Focus 6, D3 and D5 coverage: not a defect.
  D3's refused registration posting one notice and showing in `lym beam status` is carried by card 21 (`refused` wait, one notice, `View.catch_up`) and card 8 (`machine_refused`);
  D5's no `bolt-hold` entry for a run holding its parent lock is carried by card 15's `HoldsParentLock` and card 21's wiring, each with a test row.
- Focus 7, text describing the retired owner pause: not a defect.
  `pauserecord/doc.go`, `machinecli`'s `recoveries` and `PATTERN-dobby-runs.md` are under card 23's `**Edit:**`, and `PATTERN-owner-machine.md` under card 25's;
  the lines each card names match the text in the tree.
- `lens-review*.md` files sit in the plan directory outside the Card Index and carry no card; no finding is raised against them.
