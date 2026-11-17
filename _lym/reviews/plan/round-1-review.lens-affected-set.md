```yaml
lens: affected-set
outcome: reported
```

Restating the task: review the plan through the affected-set lens. For each output, behaviour or text a card changes, find what reads it, say which cards cover that set, and report any gaps.

**Per-change affected sets**

- **Verb retired; `--map` added (card 1).** Readers: README.md:77, docs/overview.md:303/831/956/970, internal/dobbyengine/doc.go:506, internal/gateslot/doc.go:67/87/177/184, internal/gateslot/template.yaml:3/7, internal/gatecli/doc.go (5, 566, 587, 720, 725, 1021–1161), internal/gatecli/redundant.go:104/129/162, pattern/PATTERN-test-speed.md:221/289, pattern/PATTERN-daemon-wakeups.md:46/54, cmd/lym/daemonwakeups_test.go:41, redundant_integration_test.go:239, prebuild.go:1, and the testmap*.go file comments. Cards 1, 3 and 5 cover all of them.
- **Refusal prefix changes from `gate testmap:` to `gate test --map:` (card 1).** Readers: the `Refusals` rows (card 1). `AGoEnvFailureEndsTheWalk` and `ACallEndedDuringABuildOrATestWritesNothing` match on substrings that stay the same. Covered.
- **`testRequest.words` gains `--map` (card 1).** Readers: `detachKeyDir`, the detached runner's replay (detachrun.go:373), `handsOff`/`flagValues` (which only read `gate`) and `TestTestRequestWords_MainPackages`. Card 1 covers them.
- **Package set filter: has a map, or tracks a Go file and holds a tier0/1 test (card 1).** Readers: `TestGateTestMap` (notests, ignored; card 1 covers these) and `TestGateTestMapLinkedPackages`, whose `linked` and `linked/inner` have no test and no map. That second set is not covered by any card (finding 1).
- **Envelope reshaped (cards 1, 3, 4: `left_out`, `passed_alone`, `failed_tests`, `unmeasured`; `skipped`, `queued` and the plan/mismatch keys removed).** Readers: `testMapOutcome`, `testMapVerbCounts`, the subtests, gatecli doc.go and docs/overview.md. Cards 1, 3 and 4 cover them.
- **Exit code becomes 1 on a failed measurement (card 3).** Readers: every subtest that decodes a run. Card 3 lists most of them but not all (finding 2).
- **`testMapSkip.Cause` renamed to `Reason` (card 4).** Readers: the uses in `TestGateTestMap` and `TestGateTestMapLinkedPackages`, including card 1's `left_out` assertion. Card 4 edits both tests. Covered.
- **testcover planning deleted (card 2).** Readers: testcover doc.go:68/80/88/94–117, gatecli/doc.go:1050–1052 (card 1) and testmapstamp_test.go:1. doc.go:68 and testmapstamp_test.go:1 are not covered (findings 3 and 4).
- **`lym` build removed and stand-in hang dropped (card 3).** Readers: prebuild.go:1 (card 3), `ACallEndedDuringABuildOrATestWritesNothing` (card 3), and redundancy_integration_test.go:235, which uses only the stand-in role and module env, so nothing breaks. Covered.
- **Slot ask, side-by-side count and memory clamp (card 3).** Readers: gateslot/doc.go:67/177/184, PATTERN-gate-slot and template.yaml:7, whose generic wording stays true. Cards 1 and 3 cover them.
- **`redundant` way-forward text (card 5).** Readers: redundant.go:104/129/162 and the `TestGateRedundant` rows. Card 5 covers them.

**Findings**

1. **BLOCKING · consistency** — card 1 (`internal/gatecli#TestGateTestMapLinkedPackages`) and card 4.
   - Card 1's new package set leaves out any package that has no map and no tier0/1 test. In the LinkedPackages fixture that is `linked` and `linked/inner`: they have no test file (testmap_integration_test.go:1372–1378) and no map in a fresh hub.
   - Several subtests in that test depend on those two packages getting maps, and no card updates them:
     - `TheMapRecordsLinkedFunctionsAndFilesTheSpawnVerdictAndAMarker` expects `Built` to include `linked` and `linked/inner` (line 1445).
     - `rebuiltInnerStamp` names `linked/inner` and expects it built (line 1435). `TheTiersDigestIgnoresTheParallelCountAndFollowsTheCommand` and `TheHubEnvDigestFollowsTheHubsVariables` use that helper.
     - `AGoEnvFailureEndsTheWalk` calls `readTestMap` on both maps (lines 1867–1868), and that helper fails the test when a map is missing.
     - `ADirtyLinkedPackageLeavesTheMapAsItWasAndCountsNothingAgainstTheCap` expects `linked/inner` built (line 1717). Card 4 even restates "linked/inner built".
   - Card 1 handled the same change for `notests` in `TestGateTestMap` but missed these. The batch's tier1 gate will fail.

2. **LOW · consistency** — card 3 (`internal/gatecli#TestGateTestMap`).
   - Card 3 says each subtest asserts the exit code it expects, and it lists the ones that should exit 1.
   - It omits two of card 1's new subtests that will exit 1 once card 3 lands:
     - `AMapRunReplacesAnExistingMapWhole`: it maps `outcomes`, and `TestOutcomesFails` fails (line 745).
     - `AnEllipsisTargetMapsEveryPackageBelowIt`: `./...` includes `outcomes`, `alpha`'s hanging test and `broken`'s failed build.

3. **LOW · consistency** — card 2 (`internal/testcover/doc.go`).
   - Card 2 deletes `Toolchain.GoRoot`.
   - Its doc edit names only the `PlanUpkeep` and `Selector` paragraphs and the `Map.FailedAt` sentence.
   - The sentence "and GoRoot its root directory" at internal/testcover/doc.go:68 would be left describing a deleted method.

4. **NIT · consistency** — card 1.
   - Card 1 removes the selector mention from `stampEnvironment`'s comment.
   - The header of internal/gatecli/testmapstamp_test.go:1 still says "the environment stamp a coverage map build and the plan's selector both compute", and no card lists that file.

Tool calls:
1. A repo-wide grep for `testmap` and for the planning names (`PlanUpkeep`, `Selector`, `GoRoot`, `FailedAt`, `.build.lock`, the envelope keys). The manifest's greps covered only selected files and didn't check spec, docs or test-file readers outside gatecli.
2. A grep for users of the reshaped test helpers and stand-in constants outside testmap_integration_test.go, plus help-text pins. The manifest hadn't checked other test files, such as redundancy_integration_test.go.
