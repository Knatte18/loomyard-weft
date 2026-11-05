```yaml
lens: row-validity
outcome: reported
```

Task: a row-validity review of the 18-card plan's test rows. I report only the rows that cannot reach what they claim, are missing, repeat an existing test, or lack a helper.

**MEDIUM: card 9 (D11 repair restart).** The planned rows cannot catch a wrong restart rule, and the rule as written allows one re-step per wake instead of "the cap's times per ten minutes".
- In `goDriver.facts`, `RepairCount` is set to zero only while the stamp is at least ten minutes past the *last* restep record. After that one restart, the driver writes a new restep record. On the next crash the stamp is older than that record, so `repairCount` counts every record for the key (three or more). That is at or above `driveRepairCap`, so the stop goes to the tackler.
- The row "stamp eleven minutes past → re-stepped again" passes, but no row drives a second crash after the restart.
- Fix: add that row, and state the rule as counting only the restep records after the qualifying stamp.
- Evidence: `internal/sheddriver/driverules.go:201` (`repairCount` counts all records), `internal/sheddriver/drive.go:727`.

**MEDIUM: card 15 (D3 refusals 3, 4 and 6).** The rows for these refusals run on fake seams that return the answer, so the judgment logic is never tested.
- "A parent with no upstream judged by its local tip", "merged with only a parent merge past it passes" and "a description finding" each get the answer straight from a fake `ParentUnmerged`, `MergedLacks` or `DescriptionFindings`.
- The logic lives in the `hubLandDeps` closures, and no row reaches it: the upstream-vs-local choice, the second-parent-reachable classification, and `ValidateDescription`.
- The acceptance criterion "a parent with no upstream judged by its local tip alone" therefore has no row that fails if that logic is wrong.
- Evidence: card 15's Intent puts all three in `hubLandDeps`, and its Tests list only `TestMarkReady` rows over `newLandWorld` fakes (`internal/beamcli/land_test.go:53-93`).

**MEDIUM: card 17 (D12).** The planned `TestStart_Refusals` row cannot reach the `ownLanding` derivation, and the table cannot check a passing start.
- The table injects `h.cli.readBeamFacts` as a fake (`internal/orchcli/start_test.go:921-929`). So a row there never runs the real `readBeamFacts`, where `ownLanding` comes from `RunHolds` and `pawlengine.LandingRows`.
- The table also only checks refusal text (`wantIn`, `wantNothingStarted`). It has no field for "orch started, dobby not started, warning reported".
- No row covers the `LandingRows` membership read either: Finalize and Beam-Down own, any other producer not.

**MEDIUM: cards 3, 13, 14 and 15.** These tests need a fake `pullrequest.API` that does not exist, and no card lists one.
- `internal/pullrequest/client_test.go` has only the scripted httptest server, and `internal/beamcli/*_test.go` has no fake API (`grep "pullrequest.API"` over beamcli tests: no hits).
- Card 3's `TestReadMergeState` "drives a fake `API`".
- `TestPROpen`, `TestPRUpdate`, `TestPRStatus`, `TestPRClose` and card 15's merge-state rows need a fake returned through `PullRequestSide`.
- The fake has to implement `Status`, `RequiredChecks`, `FindOpen`, `Create`, `Edit`, `SetDraft`, `Reopen` and `Close` with call recording, and its `plan:` handle is missing from cards 3 and 13.

**MEDIUM: card 13.** A prescribed refusal has no reaching row.
- The refusal: a record write that fails after the pull request was created is refused, naming its URL and `lym beam pr open` again.
- `TestPROpen` lists no failing `WriteLanding` row.
- The "recorded and merged names `lym beam ready`" branch also has no row.

**MEDIUM: card 8.** The wiring branches of `pawlRetiredMove` are untested.
- `ScratchPullRequest` reads a found record whose GitHub read fails as found, "so the pull request is never dropped on a transient failure". `EndSession` ends the session through `runproc`.
- Both are in loomcli, and `TestWire_BeamRecipe` only asserts the field is set.

**LOW: card 14.** "A failed mode read still closes and drops, and reports the read's error" has no `TestPRClose` row.

**LOW: card 7 (test economy).** The new top-level `TestStartReady_ReadyBeamStartsNoMember` should be a subtest of `TestStartReady_RefusalsAndBackoff`. That test already holds the sibling behaviour "a start refusal spawns no pass, notices once per entry, and starts on the first tick after it is gone" (`internal/dobbyengine/starts_test.go:144`).

**LOW: cards 7 and 16 (harness).** `newLoopHarness` needs changes neither card lists.
- It builds a prime geometry: `WorktreeName "prime"`, no `Slug` and no `AnnouncePath` (`internal/dobbyengine/loop_test.go:113-121`).
- Card 16's `TestAnnounceLandings` and card 7's beam-only hold need a beam `Slug`, and card 16 also needs `AnnouncePath`. Neither card lists the harness edit.
- Card 7 also leaves open whether `holdReadyBeam` gates on `Slug` or on a nil `BeamReady`. A row fed through this harness depends on that answer.
- Card 16 has no row for a failed read (only for a failed post).

**LOW: cards 12–14 (fixture default).** `newLandWorld` sets `ready: {"b": true}` (`internal/beamcli/land_test.go:67`), so `judgeNotReady` refuses every "passes" row unless the row clears the flag.
- Affected rows: `TestDescribe`'s "described", and `TestPROpen`, `TestPRUpdate` and `TestPRClose`'s success rows.
- No card says to clear it, or to change the default and fix `TestMarkReady`/`TestLand` instead.

**LOW: card 9 (arbiterstop test).** `TestCheck_InterruptedPawlRowsAreEligible` "read through `NewReaders`' policy" needs a `*lymcwd.Location`, because `NewReaders(location)` builds disk-backed readers (`internal/arbiterstop/arbiterstop.go:194`). A `Readers{InterruptPolicy: loomshed.InterruptPolicyFor}` pins the same thing without a hub fixture.

**LOW: card 9 (wake stamp test).** The `WriteWakeStamp`/`ReadWakeStamp` round trip is placed in `TestPathConstructors`, which is a path test. It also has no row for the absent-file not-found case the Intent states.

Further tool calls (5):
1. Grep of `internal/dobbyengine/starts_test.go` and arbiterstop tests: my context had only the test names, not the subtest shapes.
2. `arbiterstop_test.go:21-64` and `newLandWorld`: I needed the `Check` test harness and the default fixture values.
3. Grep for fake `pullrequest.API` types and the `NewReaders` signature: to check whether a fake exists for cards 3 and 13–15.
4. Grep of the dobby loop harness, the `tick` methods and `TestStart_Refusals`: to see the harness geometry and the refusal table shape.
5. `loop_test.go:100-125` and `start_test.go:848-929`: to confirm the harness fields and the held-beam row's injected `readBeamFacts`.
