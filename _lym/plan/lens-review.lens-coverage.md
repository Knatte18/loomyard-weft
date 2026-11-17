```yaml
lens: coverage
outcome: reported
```

Coverage lens: I checked whether every decision, acceptance criterion and new branch has a carrying card and a planned row that would fail if it broke.

**Findings**

1. **MEDIUM, D4 absence definition (orch watcher).** The record names the orch watcher as an observer of a sleep ("the dobby loop or the orch watcher"). The overview's Shared Decision "absences and their observers" drops it, so no card carries that observer and no row tests it. That is a waived decision with no `Added after Discussion` entry. Evidence: `_lym/discussion/decision-record.md` D4 opening line; `00-overview.md` "absences and their observers".

2. **MEDIUM, D4 (the dobby's own hand-over).** The record says "The dobby's own handover of a subject calls `pairteardown.EndSubjectProcesses` with the dobby end set". Card 21 swaps `pauseRunsForOwner` for the catch-up, which calls `EndSubjectProcesses(prime, slug, false)` per run through `HandOver`. No card calls it with `endDobby` true for the subject, and no row asserts it. Evidence: decision record D4 bullets; card 21 Intent ("`handOver` ... runs the catch-up in place of `pauseRunsForOwner`"); `internal/pairteardown#HandOver` calls `EndSubjectProcesses(prime, slug, false)`.

3. **MEDIUM, D4 step 2 (operand narrower than the record).** The record fetches "each local pair's branches". Card 21 scopes the dobby's catch-up to `DobbyPairSlugs`, which covers only runs whose parent is the dobby's worktree.
   - A beam that reads owned elsewhere at a fresh start gets no dobby, because `EnsureLive` refuses it.
   - So the local pairs of that beam's runs are never caught up or handed over.
   - Evidence: card 20 (`DobbyPairSlugs`); card 21 wiring; card 8 (`EnsureLive` refuses an elsewhere beam); `internal/dobbyengine#EnsureLive`.

4. **MEDIUM, D5 upgrade (operand differs from the record).** The record says the first read "pushes the pause intents and the Bolt hold first, and only then drops the `adopt` and `adopt-hold` entries". Card 22 step 3 instead turns the entry of each intent run "into a plain slug entry of the default caller in place". The record drops them, and the catch-up's step 5 writes entries from intents. TestUpgradeRead's row "the entry of an intent run turns into a plain entry" pins the deviation rather than the record. Evidence: decision record D5 upgrade bullet; card 22 Intent step 3.

5. **MEDIUM, D3 criterion "in `dobbyengine.OwnedWorktrees` too".** Card 8 moves the registry judgment into `hubgeom.localBeams` and `DobbyStart` (the `Elsewhere` and `PrimeOwner` flags). Its rows test only that the engine follows the flag. No row reaches the registry judgment that feeds `OwnedWorktrees`: neither a former name reading here nor an unregistered machine reading a registered current name as elsewhere. Evidence: card 8 Tests; acceptance criteria D3 #3 and #5.

6. **MEDIUM, D4 criterion "processes end … one notice naming any dropped intent".** Card 17's `TestHandOverOwnedElsewhere` asserts the pushed commit, archive tags, pair removal, origin branches and mark. It asserts neither the process end nor the notice naming a dropped pause or cancel intent. TestCatchUp works over fakes, so it cannot either. Evidence: card 17 Tests; acceptance criterion D4 #1.

7. **MEDIUM, D4 criterion "a run owned elsewhere at Finalize past its parent-side merge has its processes end here".** This changes behaviour: the deleted `EndOwnedElsewhere` skipped such a run. No row in `TestCatchUp`, `TestHandOverOwnedElsewhere` or `TestRun_CatchUp` uses a landing past its merge. Evidence: card 17 ("A `landing` run past its parent-side merge is handed over the same way"); `internal/tendercli#ownedElsewhereEnder.run`'s past-merge skip; acceptance criterion D4 #3.

8. **MEDIUM, D4 criterion "local records ahead of origin are pushed, none reset" (production seams).** No row reaches `HubCatchUp`'s production seams:
   - `Ahead`, read from the prime's records side or the pair's sides;
   - the `FetchRun` transport → `ErrOriginUnreachable` mapping;
   - the `PullBolt` skip → error mapping;
   - `PushOwed`'s `ErrHandedOver` → nil.

   `TestCatchUp` and `TestCatchUpRun` use fakes. Evidence: card 20 Tests; card 18 Tests.

9. **MEDIUM, D5 criterion "a run whose push is owed gets no `bolt-hold` entry".** Card 15 relies on the verdict reason being `push_owed`, not `held`. No row in `TestSyncBoltHold` or `TestRun_SyncsBoltHoldEntries` covers a held hub with an owed-push run, so dropping its entry is unpinned. Evidence: card 15 Intent; card 21 Tests; acceptance criterion D5 owed push.

10. **MEDIUM, D5 criterion "a subject read as owned elsewhere leaves no process of its runs, its orch or its dobby".** Card 23's bring-up calls `EndSubjectProcesses(prime, slug, false)`, so no dobby end. `TestStart_ReconcilesTheHubFirst` only asserts that the seam is called. No row asserts that the orch and the dobby are gone. Evidence: card 23 Intent and Tests.

11. **MEDIUM, card 4 `Registry.apply`'s `Forget` branch and former-name copying.** `TestRegistryCheck` tests `Check` only, and `TestRegisterMachine` has no `Forget` row. Card 11 exercises `Forget` only through `fakeHubs`, so the real removal of a forgotten name from the entry is unreached. No row asserts that a first registration copies the file's former names (D3: "copying the former names of its `machine.json`"). Evidence: card 4 Tests; card 11 Tests.

12. **MEDIUM, card 11: the production `references` reading.** The listing of owners, takeover entries and unfinished cancel intents is tested only over `fakeHubs`. No row reaches the production reader: the fetched board plus `pairteardown.ReadBattenRecord` over local pairs. The created `hubReferences(hub, name string)` also takes no `deps`, which contradicts "reads them through the deps seam `references`". Evidence: card 11 Create and Intent.

13. **MEDIUM, card 10: a failed per-hub registration.** "a failed registration is that hub's failure with the re-run as its way forward, and the other hubs go on" has no `TestName_Set` row. Evidence: card 10 Tests.

14. **MEDIUM, card 14: the pull-failure branches.** Neither `hubRunVerdict` nor `hubRunIntents` has a row for "Any other failed pull refuses, naming … the way forward" or for "unreachable reads the local Bolt and goes on". Only the hold row is planned. Evidence: card 14 Tests.

15. **MEDIUM, card 17: a transient push failure.** "A failed push for a transient reason returns the push's error … nothing is archived first" has no row. The `PushRun`-meets-takeover → `ErrHandedOver` → nil branch is unreached too. Evidence: card 17 Tests.

16. **MEDIUM, card 21: a failed catch-up in `handOver`.** "a failed catch-up keeps the dobby up and retries at the next probe tick" has no row. `TestRun_ForeignOwner` asserts only that the catch-up replaces the pause. Evidence: card 21 Tests.

17. **LOW, D5 criterion "pushes it at its first pass after the hold clears".** Card 13 plans only the waiting half, for a Finalize cut between its local merge and its parent push. The push after the hold clears has no row. Evidence: card 13 Tests; acceptance criterion D5 Finalize.

18. **LOW, D5 upgrade order.** `TestUpgradeRead` has "a failed intent push drops nothing" but no row for a failed `WriteHold` dropping nothing. It also has none for a re-run that rewrites a hold a green check cleared. Evidence: card 22 Tests; D5 upgrade bullet.

19. **LOW, D5 (queued tasks).** Card 22 limits the hold's queued-task subjects to tasks "owned here". The record says "belonged to a released `queued` task" with no owner condition. Evidence: card 22 step 2; D5 upgrade bullet.

20. **LOW, consistency between cards 6 and 10.** Card 6 says the sweep's write carries no `Identity` ("the sweep's own pre-registered write"). Card 10 says "the owner sweep, whose request carries the identity". Which branch of `writeOwners` the sweep takes is undecided. Evidence: card 6 Intent; card 10 Intent.

21. **LOW, card 10 `nextFile` with `dropCurrent`.** `nextFile` adds `cur.hostname` as well as `cur.name`. A refused name that is also the hostname-rule name would re-enter the former names unless `dropCurrent` drops both. No row covers a refused hostname name. Evidence: `internal/machinecli#nextFile` (`for _, old := range []string{cur.name, cur.hostname}`); card 10.

22. **LOW, small unreached branches:**
   - card 3: `Resolve` when a name is one entry's current name and another's former name;
   - card 4: no-op re-registration writes nothing; `EnsureIdentity` with an invalid or unreadable file;
   - card 5: same machine with a different kind or different subjects;
   - card 13: empty `HoldMachine` wording;
   - card 19: one slug's failed read is joined while the others go on;
   - card 20: scope wiring of `HubCatchUp(prime, [slug])` (`TestCatchUpRun` fakes the seam);
   - card 21: `syncBoltHold` failure warning.

   Evidence: each card's Tests section.

Tool calls made: none.
