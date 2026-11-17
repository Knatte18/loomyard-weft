# Fixer report — plan, round 1

Every finding's premise held against the code.
`lym loom validate-plan` answers `ok: true` after every change below.

## Changes

- **F1** (fixed as proposed): card 21's fourth owing trigger now needs `RunView.PairPresent`, and the Intent says why `DobbyRuns` lists runs whose pair is not here.
  `TestRun_CatchUp` gains the row where a run answers `hand_over` with `PairPresent` false.
- **F2** (fixed differently): card 15 lists `internal/tendercli#resumeAside` under Edit and gives it `Hold`, with the reason in the Intent.
  Not done: the proposed `TestResume` row whose `Verdict` seam answers `held`.
  `Resume` returns the verdict's refusal before any lift (`internal/tenderengine/resume.go:122-128`), so that row would assert a lift that cannot happen; with `Hold` set aside the verdict no longer answers `held`.
  The new row is in `TestPauseAndResume_WriteLiftAndRefuseOverTheHub` (tier1, now under Edit) instead, the one test that reaches `resumeAside` through the verb:
  under a standing hold, resuming a slug with its own entry lifts that entry and writes a `bolt-hold` entry in its place.
- **F3** (fixed as proposed, then widened by the self-check):
  - Card 8 lists `fakeStart.baseDeps`, `runAnAdoptHold` and the five loop tests that set `Deps.Owner`, plus `internal/beamcli#TestHubDobbyDeps_ABeamWithNoOwnerOfItsOwnIsUnderARootAdopt`, which calls `deps.Owner()` (`internal/beamcli/dobbydeps_integration_test.go:392`).
  - Card 23 lists `newPassHarness`, `runAnAdoptHold`, `TestRun_ForeignOwner`, `battenshed#newLockHarness` and `loomcli#stoppingPrePushCheck`, and its test lines say which field each drops.
- **F4** (fixed as proposed): card 16 drops `refuseStartOwnedElsewhere` and `TestRefuseStartOwnedElsewhere`.
  Its Intent now says `refuseByAct` refuses `lym batten run` under a hold with card 13's refusal, and that the fix task starts because `ReadRun` reads it as not held.
- **F5** (fixed differently): card 8 makes `EnsureLive` report a failed `Machine` read as an `OutcomeRefused` entry for each named worktree, with the refusal as its detail, and adds a `TestEnsureLive` row for it.
  The proposal put the notice in "DobbyStart's caller in internal/beamcli", but `EnsureLive`'s only caller is `internal/orchcli/start.go:390` (`ensureDobbies`).
  So `orchCLI.startDobbyStep` posts the refusal once per bring-up through `orchcli.NotifyPrime`.
  The proposal said `lym beam status` shows "the refused start entry", but `dobbyStatus` reads only dobby state records, which a refused start never writes (`internal/beamcli/cli.go:353`).
  Instead, status gains `machine_refused`, read through a new no-write `boardengine.IdentityRefusal`:
  - card 4 creates it as `identityRefusal(machinename.FilePath)` and `ensureIdentityWith` shares it;
  - card 8 lists `statusResult`, `dobbyStatus` and `statusCmd` under Edit;
  - card 8 creates `TestDobbyStatus_ShowsTheIdentityRefusal` (tier1).
- **F6** (fixed as proposed): card 9's `machineNamesOf` leaves out of an unregistered identity's former names every name another entry holds, matching `ReadsAsHere`.
  `TestMachineNamesOf` gains the row.
  A current name held by another entry is stated as card 4's registration refusal, together with how far that refusal reaches.
- **F7** (fixed differently): the proposal filtered the held list in `hubDobbyDeps` and tested it in `TestRun_SyncsBoltHoldEntries`, but a dobbyengine test over fakes cannot see a filter inside beamcli's wiring.
  The exemption now sits in `tenderengine.SyncBoltHold`:
  - `BoltHoldRequest` gains `HoldsParentLock`, and a held slug holding its parent lock gets no entry, with any standing `bolt-hold` entry lifted (card 15);
  - `TestSyncBoltHold` covers that row;
  - card 21 wires the seam to `parentlock.Store.HeldBy` over `parentlock.Dir` and names the exemption in its held-list paragraph.
- **F8** (fixed as proposed): card 19 re-signs `commitRunIntent` with `returnOwed bool` and moves `hubRunIntents` from Uses to Edit.
- **F9** (fixed as proposed): card 17 gains a Bound paragraph with three limits:
  - only a run whose owner reads elsewhere after registration landed is handed over;
  - the forced removal comes after the archive tags reach origin, because `HandOver` returns the archive's error first;
  - nothing rewinds or force-pushes an origin branch, and nothing writes the board or another machine's records.
- **F10** (fixed as proposed, except the adoptcli lines; see Deferred):
  - Card 23 lists `internal/pauserecord/doc.go`, `internal/machinecli#recoveries` and `pattern/PATTERN-dobby-runs.md` under Edit and `internal/machinecli/doc.go` under Uses, and says what each drops.
  - The self-check widened this:
    - the hibernate-wake half of pauserecord's sentence goes too;
    - the next recovery line loses its reference to the probe;
    - landingshed's owner-pause texts are listed: `adopterFirstRepair`, `parentGuard.release`, `parentGuard.stillHeld`, `landingshed/doc.go`, and `TestFinalize_LockRecheckAndLanding`, which pins the halt wording.
  - Card 25 names the two `PATTERN-owner-machine.md` lines it rewrites.
- **F11** (fixed as proposed): card 11 lists `pattern/PATTERN-cli-cobra.md` under Edit and adds `pairteardown` to the machinecli line.
- **F12** (fixed as proposed): the overview's decision on absences and their observers drops the false premise and says the plan runs the catch-up on every non-continue pass by choice.
  Card 20's Bound explains why a pass's catch-up racing the dobby's over one pair is safe.
  The self-check replaced the reason "a push after a hand-over stops at the hand-over mark", which `takeoverStop` does not do without a takeover entry (`internal/fabricengine/takeover.go:117-121`).
  The new reason: a pass that loses the race fails its push or hand-over and refuses, acting on no run.

## Self-check

- fix-affected-set, BLOCKING: `TestHubDobbyDeps_ABeamWithNoOwnerOfItsOwnIsUnderARootAdopt` calls `Deps.Owner` with the old shape and was not in card 8. Fixed as proposed.
- fix-affected-set, LOW: landingshed's texts still name the owner pause after card 23. Fixed as proposed.
- fix-proposal, MEDIUM: `TestRun_ForeignOwner` sets `PauseOwned` and was not in card 23. Fixed as proposed.
- fix-proposal, LOW: lines next to the F10 edits (pauserecord's hibernate half, the recovery that refers to the probe) were left stale. Fixed as proposed.
- fix-claims, MEDIUM: card 20's hand-over-mark claim does not hold on card 17's route. Fixed with my own change: the reason is replaced, as described under F12.
- fix-claims, LOW: card 9's new sentence overstated how far the registration refusal reaches. Fixed as proposed: the sentence now says hand-run fabric verbs still read that name as here.
- fix-consistency, LOW: card 14 gives a reason for tender resume's pulled verdict that no longer holds. Fixed as proposed: card 14 notes that card 15 sets the hold aside and that the pull keeps the owner reading fresh.
- fix-consistency, LOW: card 13's ImpactSummary said every act-rule caller waits under a hold. Fixed as proposed.
- fix-consistency, LOW: card 4's ImpactSummary left out `IdentityRefusal`. Fixed as proposed.

Every fork returned a report.

## Disputed

None.

## Deferred

- **F10, `internal/adoptcli/doc.go` lines 33 and 210:** these lines still name the owner pause after card 23.
  Listing the file under card 23 trips `uses-later-target`, because cards 1, 6 and 7 list it under Uses, as the review itself records.
  Before card 23 the lines are still true, so no earlier card can rewrite them either.
  Fixing them needs a change to the plan's ordering or to the validator rule, which is the plan owner's call.

## Rulings

None reached this round.
