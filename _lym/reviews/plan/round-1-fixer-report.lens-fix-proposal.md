```yaml
lens: fix-proposal
outcome: reported
```

Lens: fix-proposal. I challenged each finding the fixer fixed as proposed (F1, F3–F14 and the F2 parts it carried out) by trying to build a case where the proposed change is wrong.

**MEDIUM — Card 3, Tests, the `Collides` stand-in bullet (F12, fixed as proposed).**
Summary: shortening the wait to 300 ms (`testMapExcludeWait`) makes the side-by-side failure timing-dependent, so `TestsRunSideBySideAndAFailureBesideAnotherEndsPassingWhenRunAlone` can no longer assert that both tests are in `passed_alone`.
Evidence:
- `runsApart` keeps a test recorded for only `testMapExcludeWait`, which is 300 ms (`testmap_integration_test.go:109, 405-416`).
- Two `.test` processes started side by side collide only if the second records itself inside the first's 300 ms window. On a loaded machine the second can start later. The first then passes beside the second, the second sees no record, and neither test enters `passed_alone`.
- The existing subtest over the same `runsApart` already allows for this. Its comment at `testmap_integration_test.go:1162` reads "the side-by-side run may fail both". It asserts only the entries, never which tests failed side by side.
- Card 3 still wants "both in `passed_alone`" (D5). Under load that is the same kind of time-bound outcome `PATTERN-test-speed` bars and that F12 set out to remove. The proposal moved the margin from the alone pass to the side-by-side collision.
- `ATiersMemoryMBClampsItsTestsToOneAtATime` is not affected, because it wants `passed_alone` empty.
- Fix: pin the overlap without a wall-clock window. For example, use a `Meets`-style rendezvous where each test waits for the other's record before it fails, bounded by `testMapMeetWait`. Or assert `passed_alone` as a subset of the two tests.

**LOW — Card 3, Create list and the `Collides` bullet (F12).**
Summary: once `Collides` calls `runsApart` over a directory named by an env var, it duplicates the existing `Excludes` suffix. The new `testMapCollideEnv` and the new suffix add nothing.
Evidence: `testmap_integration_test.go:323-324` already makes an `Excludes` test run `runsApart(os.Getenv(testMapExcludeEnv), name)`. The excludes fixture at line 1153 already uses `TestExcludesFirstExcludes` and `TestExcludesSecondExcludes`. The card's renaming of the excludes package's tests to `Collides` is now a rename with no change in behaviour.

The other as-proposed fixes held against every case I built:
- **F1:** the hand-made maps keep `linked` and `linked/inner` in the `--all` set and the explicit-target set.
- **F3:** the cap kill and the guard kill each keep their own subtests.
- **F4:** `readCards` refuses before the scoper check, under any scoper.
- **F5:** a whole tier0 share maps every package.
- **F6, F7, F8, F9, F10, F11, F13, F14:** no case found where the change is wrong. For F11, `notests` tracked a source file in the old fixture (its `Built` at line 951).

Further tool calls:
1. `grep` for `Excludes`/`testMapExcludeEnv` uses and the stand-in env constants in `testmap_integration_test.go`. My context had `runsApart` but not its caller, the excludes subtest's comment, or the list of env knobs, which I needed for F12 and F13.
