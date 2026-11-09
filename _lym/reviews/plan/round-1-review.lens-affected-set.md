```yaml
lens: affected-set
outcome: reported
```

Task: the affected-set lens. For each change the cards make, list everything that reads or asserts it, and check which cards cover that set.

**Per change: the set found, and the cards that cover it**

1. **The green end only reports (D8) and the adopt hold is gone (D5).** This falsifies every text that says a green check lifts holds or starts runs.
   - Covered: `green.go`, `hubCheckDeps` (`check.go:502`), `ownerHaltReason`, `createCmd`, the beamcli/battenshed docs and `docs/overview.md:787,1570` (card 4); `deployStale`, `ownerWriteError` and `heldBeamWayForward` (card 6); `adoptcli/doc.go` and the pattern files (card 10); the `RunView` comment and dobbyengine `doc.go:158` (card 1).
   - Missed: `fixloop.go:335` (`fixReadError`), `fixloop.go:340` (`fixFilingError`), `battencli/owner.go:106`, the comment above `planPass` at `dobbyengine/passes.go:108`, and `docs/overview.md:785`.
2. **The step order changes, the owner write moves before the deploy, and adopt becomes a worker with a follow.**
   - Covered: `cli.go` `Long` (cards 6 and 8), `adoptcli/doc.go` (card 10).
   - Missed: the adopt entry of `docs/overview.md` (lines 782–790). Card 4 drops only two sentences from it.
3. **New files `adopt.json`, `adopt.lock` and `adopt.out` under `fabricengine.AdoptScratchDir`.**
   - Covered: the adoptcli doc's scratch-file list (card 10); `ReadCheckViews` and `primeMark` already skip non-directories (card 7 notes this).
   - Missed: the `AdoptScratchDir` doc comment (`fabricengine/junctionnames.go:188`, "holds the adopt check's record, output and lock") and `fabricengine/doc.go:1122` ("holds the adopt check's files").
4. **A held-back project now returns an error (card 6: "a held-back project no longer ends the call ok"), and a hub-wide record carries state between calls.**
   - Readers: `TestAdoptIntegration_SecondCloneTakesOver` at `adopt_integration_test.go:170-173`, which `Fatalf`s on that error. Its later `--drop-local` and `--keep-local` calls at 178 and 186 now resume the stopped record.
   - Cards 6 and 7 list the test as a target, but neither Intent describes this change.
5. **`liftOwnerPauses` is deleted (card 6).**
   - Readers: `slugOutcome.PauseLifted` (`adopt.go:74`, `pause_lifted`) is never set again; `adopt_integration_test.go:115` asserts it.
   - Not covered: no card removes the field or its assertion. `runningHereRun` (`adopt.go:41`) and `callerLabel` (`adopt.go:24`) also lose their last uses, and no card deletes them.
6. **The run report states change to `paused_by_adopt`.**
   - Readers: `runreport.go`, `runreport_test.go`, `cli_test.go` (card 5), adoptcli doc (card 10). Nothing outside adoptcli reads `waiting_for_check` or `held_failed_check`. Fully covered.
7. **`AdoptCaller` is read as held (card 1).**
   - Readers: `battencli/dobbyreads.go:168` copies the flag unchanged; the `cancelcmd` and `tender` pause/resume paths read the scope, whatever the caller. Covered apart from the `planPass` comment in change 1.
8. **`BeamUpOptions.SkipOrch` (card 2).**
   - Readers: `beamcli/cli.go:501`, `loomcli/pawlwiring.go:123` and `adoptcli/deps.go:139` all leave it false. Covered.
9. **`holds.json` is retired.**
   - Readers: `holds.go`, `deps.go`, `adopt_test`, `check_test` lines 501/512, the integration test, the adoptcli doc. Covered by cards 6 and 10.

**Findings**

- **MEDIUM | consistency | card 4 (method) | The card rewords part of the green-end text set and misses three refusal texts.**
  - `fixReadError` and `fixFilingError` still end with "the holds standing meanwhile" (evidence: `internal/adoptcli/fixloop.go:335`, `:340`).
  - `battencli` `owner.go`'s refusal still says `lym adopt` "holds its run until the adopt check of that subject is green, which then starts it" (evidence: `internal/battencli/owner.go:106`).
  - Fix: reword them to name the adopt pause and `lym tender resume`.
- **MEDIUM | consistency | card 4 / card 10 (method) | `docs/overview.md`'s adopt entry keeps the old order and model.**
  - It still says the deploy runs before the owner write, that "a failed deploy refuses the verb with the holds standing", and that adopt runs in process. No card describes the worker, the record or the new order there.
  - Card 4 drops only lines 787 and 1570 (evidence: `docs/overview.md:782-790`).
- **MEDIUM | consistency | cards 6 and 7 | `TestAdoptIntegration_SecondCloneTakesOver`'s forced change is not described.**
  - After card 6, the adopt-back call at line 170 returns an error, because the record ends `stopped`, and the test's `Fatalf` fails.
  - The `--drop-local` and `--keep-local` calls then resume the record's request, so their semantics change too. Neither card's Intent says what the test now expects (evidence: `internal/adoptcli/adopt_integration_test.go:170-191`).
- **LOW | consistency | card 6 | Dead outputs and constants are left behind.**
  - `slugOutcome.PauseLifted`, its `pause_lifted` key and its assertion at `adopt_integration_test.go:115` stay with no writer.
  - `runningHereRun` and `callerLabel` lose their last uses (evidence: `internal/adoptcli/adopt.go:24,41,74,1017`, `holds.go:175`, `deps.go:91`, `green.go:253`).
- **LOW | consistency | card 3 | Two fabricengine docs go stale.**
  - The `AdoptScratchDir` comment and fabricengine `doc.go` still describe the directory as holding only the adopt check's files, while card 3 puts the adopt record and lock there (evidence: `internal/fabricengine/junctionnames.go:188`, `internal/fabricengine/doc.go:1122`).
- **LOW | consistency | card 1 | The comment above `planPass` still names the hold-caller entry.**
  - Card 1 rewords `RunView.Held` and dobbyengine `doc.go`, but not this comment (evidence: `internal/dobbyengine/passes.go:108`).

**Tool calls**
1. A repo-wide grep for the changed keys and strings (`waiting_for_check`, `holds.json`, `pause_lifted`, "holds standing" and similar), to find readers outside the pre-read.
2. `fixloop_test` assertions, the `AdoptScratchDir` comment and doc, the agent-facing files naming `lym adopt`, and `docs/overview.md` 775–792: the pre-read had only overview lines 787 and 1570.
3. The `lym adopt` mentions in `orch-role.md` and the board `SKILL.md`, to see whether agent-facing text depends on the changed behaviour (it does not).
4. A grep for hold and adopt-check comments in dobbyengine, battencli, tenderengine and hibernateengine production files, to find readers of the hold the pre-read lacked.
