```yaml
lens: code-scope
outcome: reported
```

Task: trace each change in the diff to the board entry's goal points (code-scope lens).

**Decisions.** The decision record holds only the board entry under `## Goal`. Its `## Decisions` section is empty and it has no `Added after Discussion` entries. The goal points come from the entry's `## Expected`:
- **G1:** a push of the prime records side never classifies `main-weft` as ended on origin.
- **G2:** a transient classification does not fail the run at Seed-IDE.

**Per change:**
- **C1:** `recordsSideNotLanded` (`internal/fabricengine/pushanchored.go:320-340`) now calls `branchGoneOnOrigin` after the deferral check. It returns `errRecordsUnpushedPastPush` for a branch origin still holds. Only a gone branch gets the "ended on origin" message. A failed lookup returns the lookup error. Traces to G1.
- **C2:** `pushRecordsSide` (`pushanchored.go:208-236`) adds a retry loop. A records side that wraps `errRecordsUnpushedPastPush` is pushed again over `gitrepo.PushBackoffDelays()`, sleeping through `opts.BackoffSleep`. Traces to G2.
- **C3:** New `errRecordsUnpushedPastPush` sentinel (`pushanchored.go:308-311`). Forced by C1 and C2.
- **C4:** Doc comment updates:
  - `PushPairAnchored` doc, `pushanchored.go:111-113`
  - `recordsSideNotLanded` doc, `pushanchored.go:316-319`
  - package doc, `doc.go` hunk at 1132
  - the follow-up commit c0f9d3a7, which only rewraps these lines

  All forced by C1 and C2: the old docs described the behaviour that changed.
- **C5:** New subtest `PrimeRecordsCommitDuringThePushIsPushedAgain` (`pushanchored_integration_test.go:521-552`). It is the test for C1 and C2, so forced by them.
- **C6:** The test function's header (`pushanchored_integration_test.go:111-112` in the diff) replaces a duplicated line with a line describing C5's subtest. Forced by C5, since the header lists the function's scenarios.

**Per decision:**
- **G1:** built by C1, with C3, C4, C5 and C6 alongside.
- **G2:** built by C2, which retries the transient not-landed state instead of failing at once, with C3, C4, C5 and C6 alongside.

**Findings:** none. Every change traces to G1 or G2 or is forced by a change that does, and both goal points are built. No `Pattern-Departure:` trailer appears in `git log` over the range, and none is needed.

Tool calls: none. Everything came from inherited context.
