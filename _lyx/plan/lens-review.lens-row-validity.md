```yaml
lens: row-validity
outcome: reported
```

Task: row-validity lens review of the Drift plan's planned test rows.

**Findings**

1. **BLOCKING — Card 14, `wantSequenceOrder` rows.** The card's target can't carry the edit it describes.
   - The card says `wantSequenceOrder` "gains `Drift-Baseline` and `Drift-Await` as `skipped` entries". But that var is `deriveSequenceOrder()` (internal/loomrecipe/sequence_test.go:62).
   - The derivation's `default` branch appends `shedengine.Done` for every non-Bouncer, non-Parent-Align row (sequence_test.go:64-80). Adding the rows to `wantProducerTable` therefore expects `Done` where the run records `skipped`, because the fixture's `envkit.FullEnv` answers `drift.enabled` false (fixture_test.go: `env = envkit.FullEnv(t)`).
   - So `TestSequence_FullRunBlocksAtMergeIn` fails unless `internal/loomrecipe#deriveSequenceOrder` changes. No card lists it.
   - Fix: target `internal/loomrecipe#deriveSequenceOrder`, not the var, and give the Drift rows an `OutcomeSkipped` branch.

2. **MEDIUM — Card 13, `TestTeardown_CallSequence` and the capture-kill error branch.**
   - The test pins exact comma-joined call strings: `sessionEnd`, `skippedEnd` and each row's `wantCalls` (teardown_test.go:235-238). Inserting a capture-kill call between `seize` and `await` rewrites all of them, not just "gains assertions".
   - More important: the new error path runs after `seize` has taken the loop lock and written the teardown mark. The card says the error "is returned like the loop kill's", but the matching precedent is the run-lock wait's error, which returns `errors.Join(err, releaseLoopLock(false))` (teardown.go `EndSession`).
   - Without that, a capture-kill error leaves the loop lock held and the teardown mark in the pid file, which is wrong behaviour.
   - Fix: prescribe the `releaseLoopLock(false)` join. The error row should assert `release(ended=false)` in its call string, and `fakeSubstrates` needs a field that scripts the error; neither is named.

3. **MEDIUM — Card 5, `TestStartCapture` breakaway rows.** The rows "a breakaway refusal returns `ErrBreakawayRefused`" and "with `FallbackDetach`, retries detached" can't be reached on Linux.
   - `proc.IsBreakawayRefused` is always false there (internal/proc/tied_linux.go:98-101).
   - Fix: add a refusal-classifier seam on `SpawnRequest`, nil meaning `proc.IsBreakawayRefused`, so a tier0 fake can trigger both branches.

4. **MEDIUM — Card 7, `TestCapture_OutlivesCaller` fixture.** The row writes only a status blocked at `Drift-Await`, but the verb's own wiring also needs:
   - the run's seed, for the recipe flag `VerbStatus` reads;
   - the origin record's parent branch, for the branch-point check in `CheckMainCapture`.
   The card lists neither. Its second row, "a second run after a failed capture captures again", also needs a failing capture first, so the hub `drift.yaml` command must change between runs or be scripted to fail once. The card leaves that unstated, so the row's input isn't shown to reach the failed state.

5. **MEDIUM — Card 11, `TestDriftVerify` harness.** The rows depend on a `shedfake.SeatRunner` `RunFn` that "evaluates the table's gate entries in order as the run loop does at each turn end, running scripted agent actions between arrivals".
   - `shedfake.SeatRunner` has only scripted results and the `RunFn`/`ProbeFn`/`ResumeFn` hooks (internal/testkit/shedfake/shedfake.go:298-323). No gate-loop emulation exists, so the re-prompt, `Terminal` and attempt-exhaustion rows rest on a new test helper.
   - The card lists no such helper, and nothing states how the helper builds `chair.Gate`'s `GateOutcome`, which the producer reads to tell the compare's terminal failure from an exhausted dirty or verify entry.

6. **LOW — Card 2, `TestScopeRecord_Unruled`.** A new top-level test for a one-line filter. `ScopeRecord.Unruled` is reached by `TestDriftVerify`'s full-scope decision-halt rows (card 11), so this is a per-symbol test the rubric's "Test economy" flags. Fold it into the record tests or drop it.

7. **LOW — Card 3, `fixedGateLists`.** Its doc comment says "the eight gate keys" (internal/testkit/envkit/envkit.go:215). Adding `drift-verify` makes that count wrong, and the card doesn't name the comment.

**Further tool calls**
- Grepped for `TestAgentActivity_LiveLockWaitReadsAsActivity`, `fakeSubstrates.teardown`, `TestTeardown_CallSequence`, `wantSequenceOrder`, `hubforge.NewHub` and `lyxbin.Build`: my context lacked the existence and bodies of the tests and kits the cards name.
- Read `sequence_test.go:20-95` and the `buildSequenceFixture` head: to see how `wantSequenceOrder` is built and which `Env` answers `drift.enabled`.
- Read `fixture_test.go` env lines, the `shedbuild` `SettingKnown` refusal and `TestDarnRouting`'s assertions: to confirm the fixture fills `Setting` and `SettingKnown` and check the darn routing rows.
- Read `TestTeardown_CallSequence` and `fakeSubstrates.teardown`: to check how the new seam's call order is asserted.
- Grepped `internal/lock` for the lock primitive: to confirm a lock held in-process blocks a probe from the same process, which several "lock held by the test" rows rely on (flock does, so no finding).
- Grepped tests asserting rendered gate-key lists, then read the `TiersValue` exact-string test at `config_test.go:690-716`: to check whether adding `drift-verify` to `goReadGates` breaks a pinned string (it does not).
