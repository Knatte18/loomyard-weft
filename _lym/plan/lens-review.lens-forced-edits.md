```yaml
lens: forced-edits
outcome: reported
```

Forced-edits lens: what each card's change forces outside the members it names.

**Findings**

1. **BLOCKING — Card 3, D7 exit.** `TestGateRedundant` breaks under card 3, and no card from card 3 onward lists it.
   - Card 1 changes `TestGateRedundant` to build cov's map with `test --wait --map cov`. Its fixture holds `TestFails` (`internal/gatecli/redundant_integration_test.go:134`), which always fails.
   - Card 3 makes `runTestMap` return 1 when an entry records a failed outcome. The subtest then stops at `redundant_integration_test.go:239-240`, which fails the test on any non-zero exit.
   - Fix: card 3 lists `internal/gatecli#TestGateRedundant` and expects exit 1 from the map build, or checks the map instead of the exit code.

2. **BLOCKING — Card 3, no lym build.** The `WhileTheLymBuildHangs` row can no longer reach its hang.
   - The row sits in `TestGateTestMap`'s `ACallEndedDuringABuildOrATestWritesNothing` (`testmap_integration_test.go:1254-1256`). It sets `prebuilt_lym: true` (line 1242) so the call builds `lym` before any test.
   - Card 3 deletes `testMapRun.buildLym`. The stand-in never records the hang, so `pollUntil` times out after 30 seconds and the row fails.
   - Card 3's test bullets rework the other `prebuilt_lym` subtest into `AMapRunBuildsNoLym`, but they leave this row out.
   - `testMapHangLymBuildEnv` is read only by `runTestMapStandInLymBuild` and set only by this row, so it becomes a dead fixture switch.
   - Fix: card 3 drops the row, the `prebuilt_lym` setup at lines 1242-1243, and `testMapHangLymBuildEnv` with its stand-in branch.

3. **MEDIUM — Cards 1 and 3 against D1's "no card fixer".** A detached `--map --card` call that exits non-zero can still be handed to a cold card fixer.
   - Card 1 sends map calls through the detached runner, keyed on `--map`. `detachRun.runWithFixer` (`internal/gatecli/detachfixer.go:108-125`) calls `fixAndRerun` when all of these hold:
     - the verb is `test`;
     - `seams.cardFixer(request)` is on;
     - the exit is non-zero;
     - the stderr file holds text.
   - After cards 3 and 4 a map run exits 1 on any failed test, failed build or unmeasured package, and it writes stderr text: the memory guard's kill lines, and card 3's alone-rerun log.
   - No card lists `internal/gatecli#detachRun.runWithFixer` or the `cardFixer` seam. The Bound in card 1's Intent ("runs no … card fixer") is true only of the in-process path.
   - Fix: a card lists `detachRun.runWithFixer`, or the seam that decides `on`, and turns the fixer off for a request whose args carry `--map`. A test row with a red detached `--map --card` call should check that no fixer spawns.

4. **MEDIUM — Card 4's `unmeasured` key.** The card changes `testMapSkip`'s JSON key and rewires the skip path, but lists neither member.
   - The Intent prints `unmeasured` as "the existing `testMapSkip` record, its JSON keys `package` and `reason`". Today the type's tag is `json:"cause"` (`internal/gatecli/testmap.go:130-133`), so this renames a field and its tag.
   - It also turns `testMapRun.skip` (`testmap.go:633-635`) into an append to `unmeasured`.
   - Card 4's Edit list names neither `internal/gatecli#testMapSkip` nor `internal/gatecli#testMapRun.skip`. Both are callers or holders of the changed shape.
   - Fix: list both under card 4's `**Edit:**`.

5. **LOW — Card 1 file headers, and card 4's `testmaptree.go` header.** Comments made wrong by the changes have no target that covers them.
   - These headers become wrong:
     - `testmap.go:1-4` (plans the packages at HEAD, then measures the queued ones and audits)
     - `testmappackage.go:1-9` (queued package, "for `lym gate testmap`", re-measures the kept entries)
     - `testmapplan.go:1-3` (plans, building nothing)
     - `testmaptree.go:1-2` ("for `lym gate testmap`", and which files changed between two commits, which card 4 rewrites)
   - Card 1 and card 4 state these edits in their Intents, but no target covers a file header. Each card lists members of those files, and adding the file self glyph beside them is refused by `redundant-file-target`.
   - Report only, with that check named as the constraint. One option is to have the card list every member of a file whose header it rewrites, or to state the header edits explicitly as part of the named members' work.

6. **LOW — Card 1 test helper doc comments.**
   - `startTestMapCLI` (`testmap_integration_test.go:134-135`) says the child "runs `gate testmap` with args".
   - `runTestMapCLIHelper`'s comment (line 123) and `helperRoleTestMapCLI`'s (line 49) say it runs `lym gate testmap`.
   - Card 1 lists `runTestMapCLIHelper` but not `internal/gatecli#startTestMapCLI` or `internal/gatecli#helperRoleTestMapCLI`, whose comments go stale.
   - Fix: add them to card 1's `**Edit:**`.

7. **LOW — Card 1, `pauseFor`.** `testmapwait.go` goes whole, and with it `pauseFor`. A grep found no caller outside the deleted files, so there's nothing to fix. The point is only that card 1 deletes the file without naming the member.

Tool calls:
- A grep for callers of the reshaped helpers, types and `pauseFor`, help pins, and a `lym quarry resolve` of the plan's Uses and Delete glyphs. The context had no caller search outside the testmap files.
- A grep of `detachrun.go` and `redundancy_integration_test.go` for the fixer path and `lym` build fixture uses. The context lacked the runner's fixer trigger.
- A read of `detachrun.go:280-339`. The context lacked `runDetached`'s red-request fixer handoff.
- A grep of `detachfixer.go` for `runWithFixer` and `holdsFailureReport`. The context lacked the fixer's trigger conditions.
