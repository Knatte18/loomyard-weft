```yaml
lens: row-validity
outcome: reported
```

Lens: row-validity. I checked whether each planned test row reaches the branch it claims, fails when the code is wrong, and is needed.

**MEDIUM — Card 1, D1 detached call: the new top-level test should be a row of the existing words test.**
- `TestTestRequestWords_Map` repeats `TestTestRequestWords_MainPackages` (`internal/gatecli/detachfront_test.go:450-470`). That test is a table over the same `testRequest.words` with rows `Set` and `Unset`.
- A `--map` row belongs in that table, with the test renamed to cover every flag word (a `**Rename:**` pair on the card).
- The planned "the two rows' `detachKeyDir` keys differ" assertion is redundant. `detachKeyDir` hashes the words, so once the words differ the keys differ trivially, and the assertion can't fail independently of the words assertion.

**MEDIUM — Card 1, acceptance criterion "a detached `--map` call and a plain call … get different detached runs, and the detached map call writes the map": no row reaches the second half.**
- No planned row runs a detached (non-`--wait`) map call through `runFront` or the runner's replay (`detachrun.go:373`, `test --wait` plus the words) and checks that it writes a map.
- The words row only shows the flag sits in the key. Nothing shows the replayed runner reaches `runTestMap`.

**MEDIUM — Card 3, D4 "the memory guard kills a test and it is named": the row has no stderr to read.**
- `ATestTheMemoryGuardKillsIsNamedAndTheRunGoesOn` asserts the guard's stderr line.
- The helpers it would use (`internal/gatecli#runTestMapIn`, `internal/gatecli#callTestMapCounts`, and the `request` closure in `TestGateTestMap`, `testmap_integration_test.go:824-830`) build a `testMapRequest` with no `stderr`. Per `testMap.go:71`, a nil stderr prints no guard line.
- The card lists no helper that gains a stderr buffer, so the row as written cannot read the line it asserts.

**MEDIUM — Card 3, D5 alone rerun: `passed_alone` holding both excludes tests is timing-dependent.**
- `TestsRunSideBySideAndAFailureBesideAnotherEndsPassingWhenRunAlone` is planned to want both excludes tests in `passed_alone`.
- The existing comment says the side-by-side run "may fail both" (`testmap_integration_test.go:1162`). An Excludes test fails only when another test records itself inside its 300 ms window (`runsApart`, `testmap_integration_test.go:406-418`).
- If one process starts late, both pass side by side and `passed_alone` is empty, so the exact assertion flakes.
- A stand-in suffix that fails whenever another test of the run has started would make the row deterministic.
- The same weakness touches `ATiersTestsRunBeforeTheNextTiers` and `ATiersMemoryMBClampsItsTestsToOneAtATime` in the other direction. They want `passed_alone` empty, so a wrong implementation whose two tests happen not to overlap would still pass. Those rows can miss a defect, but they don't flake.

**MEDIUM — Card 4, D6 checks before a failure is written: two of the four prescribed branches have no reaching row.**
- The card prescribes a check over the package's own tree before a test or spawn scan failure is written, and a check over the linked trees before a failed build's failure is written.
- No row puts a dirty file beside a failed scan or a failed build. `ASpawnScanFailureWritesAFailedMapWithTheMarker` and the broken-build rows run over files the checks find clean.
- If either check is dropped, no test fails.

**LOW — Card 4, `APackageWithNoMappedTestIsCheckedBeforeItsWrite`: no cleanup is stated.**
- The row leaves an untracked file in `emptied`. `TestGateTestMap`'s subtests share one hub and run in sequence (`testmap_integration_test.go:711-713`).
- The card doesn't say the row removes the file, so every later `--all` subtest would list `emptied` as unmeasured and exit 1, against its stated outcome.

**LOW — Card 3, `AMapRunWritesNoRunRecordAndAGateCallAfterItRunsItsTiers`: its setup must keep the tree clean, and the card doesn't say so.**
- The follow-up `--wait --tier 0 steady` call runs tier0's command `x`, which fails (gate config at `testmap_integration_test.go:726`). So "not carried" is the only assertion that can catch a green record written by the map run.
- That holds only if the tree is clean enough for a green record to apply. The shared hub keeps the untracked `dirty/extra.go` (`testmap_integration_test.go:781`), and the card plans no `dirty` row removal.
- Over a dirty tree no green record applies, so the assertion passes whatever the map run wrote.

**LOW — Card 1, `TheRetiredVerb` row: its exact expected text is unconfirmed.**
- The `Refusals` loop compares `envelope.Error` with exact equality (`testmap_integration_test.go:897`).
- `clihelp.GroupRunE` returns `unknown subcommand "testmap" for "gate"` (`internal/clihelp/exec.go:235`), but the envelope `RunRoot` writes may wrap it.
- The card fixes the exact text without that being confirmed. A contains check would be safer.

Calls made:
1. Read `TestGateTest_RefusesBeforeResolvingTheHub` and `TestTestRequestWords_MainPackages`: I needed the shapes of the tests that cards 1's rows extend, and my context lacked them.
2. Grepped the gateslot config validation, `Pool.AcquireSlots` and `clihelp.GroupRunE`: I needed to confirm the refusal inputs are admitted and the retired verb's error text, which my context lacked.
