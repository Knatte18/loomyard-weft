# Lens review

Every lens fork returned a report; each is saved beside this file as `lens-review.lens-<lens>.md`.

## coverage

- C1 (MEDIUM) orch watcher as a sleep observer — disputed, premise false: D4 reads "a sleep a long-lived lym process observes through `pauserecord.SleepWatch` ... (the dobby loop or the orch watcher)", naming the processes that may observe; the plan observes every absence through the dobby loop, which `orchCLI.beamUp`'s `ensureDobbies` (`internal/orchcli/start.go`) starts beside every orch watcher, and the orch watcher (`internal/orchengine`, outside this part's packages) has no sleep watch to carry it (`internal/orchcli/watch.go:157-193`).
- C2 (MEDIUM) dobby's own hand-over without the dobby end — fixed as proposed: card 21's `handOver` calls the new `EndSubject` seam, `EndSubjectProcesses(prime, owner, true)`, after the catch-up, with a `TestRun_ForeignOwner` assertion.
- C3 (MEDIUM) catch-up scope narrower than "each local pair" — fixed differently: rather than every dobby fetching every pair, card 20's `DobbyPairSlugs` adds, for the prime's dobby, every local pair whose parent beam reads owned elsewhere (no dobby starts for it), so each local pair is caught up by one dobby.
- C4 (MEDIUM) upgrade converts intent entries instead of dropping them — fixed as proposed: card 22 step 3 drops every `adopt`, `adopt-hold` and `owner` entry, and the catch-up's step 5 writes entries from the pushed intents.
- C5 (MEDIUM) OwnedWorktrees' registry judgment unreached — fixed differently: card 8 extracts `hubgeom.beamPair` with a tier0 `TestBeamPair` holding the D3 rows, instead of a hub-level test of `localBeams`.
- C6 (MEDIUM) hand-over's process end and notice unasserted — fixed as proposed: card 17's test asserts the registered process ended and the notice naming the owner and the dropped pause intent.
- C7 (MEDIUM) Finalize past merge owned elsewhere — fixed as proposed: rows in card 17's test and card 18's `TestCatchUp`.
- C8 (MEDIUM) `HubCatchUp` production seams unreached — fixed as proposed: card 20 adds tier1 `TestHubCatchUp` (push of records ahead, one-slug scope, unreachable mapping).
- C9 (MEDIUM) owed-push run under a hold — fixed as proposed: `TestRun_SyncsBoltHoldEntries` row in card 21.
- C10 (MEDIUM) bring-up leaves orch and dobby — fixed differently: card 23 passes the dobby end for a beam entry and `TestStart_ReconcilesTheHubFirst` asserts the flag; the end itself is `EndSubjectProcesses`' own, not re-tested through a live orch.
- C11 (MEDIUM) `apply`'s `Forget` and former-name copy — fixed as proposed: card 4's `TestRegistryCheck` rows assert `apply`'s result.
- C12 (MEDIUM) production `references` reader — fixed as proposed: card 11 declares the `references` seam with `hubReferences` as its filler and adds a `TestName_SweepsHubs` row.
- C13 (MEDIUM) failed per-hub registration — fixed as proposed: `TestName_Set` row in card 10.
- C14 (MEDIUM) pull-failure branches of the verdict verbs — fixed differently: card 14 moves the pull into `pairteardown.ReadRunPulled`, whose tier1 test reaches the unreachable and dirty branches (see R3).
- C15 (MEDIUM) transient push failure and takeover answer in card 17 — fixed as proposed: rows added.
- C16 (MEDIUM) failed catch-up in `handOver` — fixed as proposed: `TestRun_ForeignOwner` assertion in card 21.
- C17 (LOW) Finalize push after the hold clears — fixed as proposed: card 13's `TestAct` pairs the held row with the same inputs unheld acting.
- C18 (LOW) failed `WriteHold` and re-run rows — fixed as proposed: card 22 rows.
- C19 (LOW) queued tasks limited to owned here — fixed as proposed: card 22 takes every released queued task an entry names.
- C20 (LOW) cards 6 and 10 disagree on the sweep's identity — fixed as proposed: card 6 now says the sweep carries the identity.
- C21 (LOW) refused hostname name re-enters former names — fixed as proposed, merged with W3: card 10's `nextFile` takes the refused names.
- C22 (LOW) small unreached branches — fixed as proposed: rows added in cards 3, 4, 13, 19, 20 (`TestHubCatchUp` scope) and 21; card 5's same-machine row is left to its idempotence row, since a hold of the same machine with other subjects is appended as the Intent says.

## row-validity

- R1 (BLOCKING) zero `Upgrade` keeps beam dobbies owed — fixed as proposed: card 22 states a zero `Upgrade` skips the first read, with a `TestRun_CatchUp` row; the upgrade is also filled for every dobby (G2).
- R2 (MEDIUM) zero `CatchUp` and `SyncBoltHold` meaning — fixed as proposed: card 21 states a nil `PullBolt` means no catch-up and a nil `SyncBoltHold` syncs nothing, with a row.
- R3 (MEDIUM) tier0 loom row cannot reach the pull — fixed differently: rather than a new tier1 loomcli test, card 14 puts the pull in `pairteardown.ReadRunPulled`, reached by tier1 `TestReadRunPulled` with a second Bolt clone writing the hold; the planned loomcli row is dropped.
- R4 (MEDIUM) tender row lacks a second clone and a run — fixed differently: same move as R3; the tendercli row is dropped.
- R5 (MEDIUM) `runFakes` not listed — fixed as proposed: cards 7 and 13 list `runFakes` (and card 7 `runFakes.reader`).
- R6 (MEDIUM) batten identity path unplanned — fixed as proposed: card 7 adds `ownerSeams.Identity`, `hubOwnerSeams`, `thisMachine`, `fakeOwners` and `fakeOwners.seams`, with a nil seam reading the name alone.
- R7 (LOW) duplicate registry rows in `TestOwnersEffective` — fixed as proposed: card 7 keeps only the empty-owner row there.
- R8 (LOW) `TestCatchUpRun` should be rows — fixed as proposed: card 20 folds the rows into `TestRunCmd_ResumeDispositions`.
- R9 (LOW) `TestSetOwners` refusal row has no registry seed — fixed as proposed: card 6 seeds through `RegisterMachine` from the second clone.
- R10 (LOW) held landing row near `HeldWaits` — fixed as proposed: card 13 states `State: Landing`, `PastParentMerge: false`.

## forced-edits

- F1 (BLOCKING) `TestResume` asserts the old held wording — fixed as proposed: card 13 lists `internal/tenderengine#TestResume`.
- F2 (MEDIUM) missing re-sign arrows — fixed differently: card 7 re-signs only the helpers with no deps value (`pawlRecheckLines`, `localOwnerVerdict`, `runOwnerVerdict`, `archiveBeforeOwnerRefusal`, `beamUpRefusal`) with their callers and tests; the deps-taking adoptcli and beamcli functions keep their signatures and read a new `Identity` deps seam, which avoids re-signing their whole call chains (`stepPause`, `stepOwner`, `createSteps`, `judgePullRequestVerb`).
- F3 (LOW) `beamcli/doc.go` under Uses — fixed as proposed: moved to card 23's Edit.
- F4 (LOW) `loomcli/doc.go` under Uses — fixed as proposed: moved to card 23's Edit.
- F5 (LOW) owner-pause readers left standing — disputed, premise false: legacy `owner` entries stand in the pause record until card 22's first read drops them, so `pairteardown.OwnerPauseHeld`, `Coverage.OwnerHeld` (`internal/pauserecord/record.go:94-96`) and battenshed's release under an owner pause (`internal/battenshed/innerrun_parentlock.go:36`) still read real entries and their comments stay true; D5 retires the caller's writers, not its readers.
- F6 (LOW) `RunView.Held` doc — fixed as proposed: card 20 rewrites it.
- F7 (LOW) machinecli doc stale from card 4 — fixed as proposed: card 4 edits `internal/machinecli/doc.go`.

## guard-reach

- G1 (BLOCKING) a run taken over while its pass is in flight is never handed over — fixed as proposed: card 21 owes the catch-up when `drivePasses` lists a run whose verdict answers `hand_over` and on a `PassExitOwnedElsewhere` exit, with a `TestRun_CatchUp` row; card 23 states the pre-push stop relies on it.
- G2 (MEDIUM) upgrade only on the prime's dobby — fixed differently: card 22 fills the upgrade for every dobby and serializes it with a lock beside the pause record, instead of moving it elsewhere.
- G3 (MEDIUM) upgrade judges a stale Bolt — fixed differently: D5 places the first read before the catch-up, so card 22 keeps that order and has `UpgradeRead` fetch Bolt itself first, an unreachable origin writing nothing.
- G4 (MEDIUM) copied-id way forward refuses itself — fixed as proposed, merged with W2: card 10 judges the id first and runs `Check` with the id it will write.
- G5 (MEDIUM) entries of runs left out of the listing — fixed differently: card 21's sync scope is `DobbyPairSlugs`, not the listed runs, and card 15 lets a resume lift a `bolt-hold` entry once no hold stands.
- G6 (LOW) sweep identity contradiction — fixed as proposed, merged with C20.
- G7 (LOW) "pushes nothing first" branch has no production writer in this part — disputed, every fix worse: the branch is D4 step 4's "when origin's record holds no takeover entry", so dropping it breaks the decision, and adding a production takeover writer is adopt's scope (decision record, Added after Discussion, split review F1).

## way-forward

- W1 (BLOCKING) the upgrade's hold has no clear in this part — disputed, every fix worse: D5 decides the first read writes the hold and that "a hold stands until a green check clears it" through adopt (D6); the split review assigns the clear to adopt-foreground, which takes this part's hold types (decision record, Added after Discussion, split review F1 and F2); every change either drops D5's hold write or builds D6's clear in this part, overturning the split.
- W2 (BLOCKING) copied-id loop — fixed as proposed: card 10 runs `Check` with the fresh id.
- W3 (BLOCKING) `nextFile` re-adds a refused hostname name and forgotten names — fixed as proposed: card 10's `nextFile` takes `refused` and never adds a refused or forgotten name, with rows.
- W4 (MEDIUM) other catch-up failures block silently — fixed differently: card 21 records a third wait reason, `failed`, with its detail and one notice per episode naming the cause and that the dobby retries; card 20 refuses a pass naming the error and the re-run; per-run failures still hold the worktree, since D4 has the catch-up run before any act rule answer.
- W5 (MEDIUM) upgrade intent push meets a takeover — fixed as proposed: card 19 returns `ErrHandedOver` unchanged and card 22 reads such a run as owned elsewhere and drops its entry.
- W6 (MEDIUM) tender resume refusal with no dobby and a stale Bolt — fixed as proposed: card 15's refusal names `lym beam start`, its `BoltHold` seam pulls Bolt first, and a resume lifts a `bolt-hold` entry once no hold stands.
- W7 (MEDIUM) `CoveredError` wording gap — fixed as proposed: card 12's wording names the dobby's lift and `lym beam start`.
- W8 (LOW) no config directory and the dobby's environment — fixed as proposed: card 4's way forward adds the dobby restart, card 21's refused notice repeats it, and card 8 fills `StartDeps.Machine` from `EnsureIdentity` so no dobby starts.
- W9 (LOW) loom pre-push stop with no pause — fixed differently: card 23 states the stop fails the run's step as today and the parent dobby's catch-up, owed by the `hand_over` verdict (G1), ends its processes; no new notice dedupe is added.
- W10 (LOW) `reconcileResume`'s unreachable branch — fixed as proposed: card 20 states it is reached only when origin is lost between the catch-up and the reconcile.
