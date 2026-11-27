```yaml
lens: code-scope
outcome: reported
```

Task: code-scope lens. I traced each change in the diff to the amended board entry's goal points and its `Added after Discussion` entries.

**Decisions**
- **G1** (Goal, "What to do"): make the healthy fixture tests insensitive to the per-test limit under load. The entry offers two ways: separate the limit the hanging test must exceed from the one the others run under, or widen the margin.
- **A1** (Added, parent, "No reproduction run on the base"): fix the flake from the code, and make the test or the code deterministic.
- **A2** (Added, parent, "No test seam"): the fix touches only `internal/gatecli/testmap_integration_test.go`, and no test seam is required.

**Changes → decisions**
- **C1**: the stand-in gains the `HangsOnCue` suffix, with its doc line (diff hunks @233 and @299; `testmap_integration_test.go:236`, `:303-309`). Traces to G1, the separation route, and to A1.
- **C2**: alpha's hanging fixture becomes `TestAlphaHangsOnCue`, and only the `--all` call sets the cue (`:857-860`, `:1064-1083`). Traces to G1 and A1.
- **C2a**: `callCounts` takes an `env` parameter (`:931-935`). Forced by C2, since the cue has to reach that one call.
- **C3**: `NamedPackageIsRebuiltWhenFresh` and `AMapRunBuildsNoLym` now expect a clean call, and the `AnEllipsis` comment is updated (`:1139`, `:1336`, `:1231`). Forced by C2, because alpha no longer times out in those calls.
- **C4**: the named constant `testMapSecondUnit` (500 ms) replaces every 150 ms unit in this file, and the gateYAML comment changes with it (`:659-663`, `:838`, and the request sites). Traces to G1, the widen-the-margin route.
- **C4a**: commit `eaa4d5228` re-breaks the `testMapSecondUnit` comment. Forced by C4.

**Decisions → changes**
- G1: built by C1, C2 and C4.
- A1: partly built by C1 and C2; see finding 1.
- A2: built. Every hunk is in `internal/gatecli/testmap_integration_test.go`, per diff piece `001-internal-gatecli.diff`, and no production file changes.

No change lacks a trace: everything maps to a decision or is forced by a traced change.

**Findings**

1. **MEDIUM, scope.** `internal/gatecli/testmap_integration_test.go:1062-1072` (`AnAllCallMapsEveryPackageWithATestAndRecordsAFailedBuild`). A1 asks for a deterministic test, but this subtest is still a timing margin.
   - What happens: this call sets the cue, so the hanging test still runs beside `TestAlphaTwo` in tier1 under parallel 2, with gateYAML's `parallel: 2` applied to tier1 (`:843-844`, `:853`).
   - Why it can still flake: a healthy test that fails or times out beside another is rerun alone and listed in `passed_alone` (`internal/gatecli/testmappackage.go:296-308`). If a stand-in start ever passes 10 s, this subtest produces the same unexpected `PassedAlone` entry the issue reports. Its `wantOutcome` compares `PassedAlone` (`:949-953`, `:1068-1072`).
   - In short, A1's "deterministic" holds for every other alpha call but not this one.
   - Possible fix: keep a deterministic route. One way is to give the call pinning the timeout a fixture package whose only tier1 test is the hanging one, so no healthy test runs beside it. The other is to state why the 10 s margin satisfies A1.

Tool calls: none. My inherited context already held the diff, the decision record, the whole test file and `runBatch`.
