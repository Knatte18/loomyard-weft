```yaml
lens: code-scope
outcome: reported
```

Task: trace each change in the diff to the decision record's goal points and its added entry (code-scope lens).

**Decisions (goal points of the board entry, plus the added entry):**
- **G1:** run the three reports concurrently with the tier 2 tests, or only on the last verify gate before a handoff, whichever the code supports cleanly.
- **G2:** keep red-first where it is.
- **G3:** the Bounds item, time one verify gate before and after and state both wall times. The added entry A1 drops it.
- **A1** (Added after Discussion, parent): no before-and-after verify timing is made in this run.

**Per change:**
- **C1.** `internal/gatecli/run.go`: `reportStepsOf`, `lastTierStart`, starting the chain at the first run of the highest tier when `code == 0`, cancelling on a later failure, the deferred `stop`, and `wait` versus a red re-judge after the lints. Traces to **G1**, the concurrency branch. For `darn-write`, whose highest tier is tier2 (summary.md "What changes"), "highest tier" is tier 2.
- **C2.** `internal/gatecli/reports.go`: `judgedReports`, `reportChain` and the held output buffers. Traces to **G1**, as the machinery C1 needs.
- **C3.** The `run` split into `judgeReport`/`writeReport` in `reach.go`, `redundancy.go` and `mutation.go`, with the call-site updates in `reach_integration_test.go`, `redundancy_integration_test.go` and `mutate_integration_test.go`. Forced by C2, so the background chain can judge without writing.
- **C4.** `memoryGuard.foreign`, `withoutForeign`, `runningCommands`, the `testRequest.commands` field in `cli.go:228`, and `commands.add`/`remove` in `tierrun.go:532-558`, with the `TestMemoryGuard` row. Forced by C1: the steps' guard samples `os.Getpid()`'s tree (`reports.go:93`), which now holds the last tier's test binaries.
- **C5.** `doc.go` "Report steps" and memory guard paragraphs, the `reach.go`/`redundancy.go` file headers, `gateslot/template.yaml:30` and `gateslot/tiers.go:118`. Forced: each documents the timing change C1 makes.
- **C6.** `reports_test.go` (`TestReportChain`) and the subtest rename in `test_integration_test.go:1537`. Forced: tests of C1/C2.

**Per decision:**
- **G1:** built by C1 and C2, with C3 to C6 forced. The concurrency alternative was chosen; summary.md "Design decisions" gives the reason.
- **G2:** no change, which is what G2 asks. The diff touches neither `redfirst.go` nor any red-first call site (diff stat).
- **G3:** no change. A1 withdraws it, so it is no finding.
- **A1:** honoured. No commit message or summary line states a measured before or after wall time of this change. The 75 to 105 s in summary.md and the "one to two minutes" in commit ec380493c restate the research doc's figures.

**Findings:** none. Every change traces to G1 or is forced by a traced change, and every decision is built or settled.

Outside my lens: `pattern/PATTERN-test-speed.md` still says the redundancy step "runs after the key's tiers released their slots" (line 383), and that reach and redundancy run "after its lints" and "after its tiers and lints" (lines 487 and 502). The diff leaves them unchanged. That stale rule document belongs to the affected-set lens.

Tool calls: none; my inherited context held everything the trace needed.
