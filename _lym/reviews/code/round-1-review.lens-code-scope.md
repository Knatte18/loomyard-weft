```yaml
lens: code-scope
outcome: reported
```

Task: trace the diff's changes to the amended board entry's goal points (the code-scope lens).

**Decisions.** The record is a decision record on its own. Its `## Goal` holds three Idea bullets. It has no `Added after Discussion` entries.
- **I1:** the parent-write audit exempts Master's own friction note path, the one its prompt names.
- **I2:** `accept-audit` accepts a finding whose path is on the run's records side and is absent or unchanged.
- **I3:** a verify fixer's commit past the last batch head does not block `accept-audit`.

**Per change:**
- **C1 builds I1.** Files: `CheckParent` gains `notePath` (audit.go), `State.MasterNotePath` (state.go), the save at spawn in `Run` (runlevel.go:805), and the callers in recordbatch.go:413 and runlevel.go:1085. Tests: `TestCheckParent` rows, plus the call updates in acceptbatch_test.go and auditseverity_test.go. Docs: doc.go:127 and the ClassParentWrite comment.
- **C1a, rename of `isOwnReportWrite` to `isWriteOf`.** Forced by C1, which reuses the helper for the note path.
- **C1b, Master stencil's "What you never do" rule (webster-template-master.md:316-317).** Forced by C1: the rule would otherwise forbid the write that I1 now allows.
- **C2 builds I3.** It covers:
  - `FixedHead` (state.go) and the gate's `recordPass` / `recordGatePass` / `mutateGateState` (verifygate.go).
  - `reconcileAcceptHead` (auditledger.go), with the reset clearing `FixedHead` (webstercli/reset.go:152).
  - Docs: doc.go:147 and :252, and the accept-audit help clause.
  - Tests: `TestAcceptPendingAudit_AcceptsVerifyFixCommits`, `TestGateStateWrites` and the fixed-head assertion in `TestVerifyGate_DirtyTreeRecordsPreFixHead`.
- **C3 builds I2.** It covers:
  - `recordsSideStatus`, `splitRecordsSidePaths` and `recordsRestoreStep` (findingtext.go).
  - The split in `AcceptPendingAudit` (auditledger.go) and in `pendingPathsWayForward` (runlevel.go).
  - Docs: the help sentence, doc.go:275-276 and the findingtext.go file header.
  - Tests: `TestAcceptPendingAudit_RecordsSidePath`.
- **C4, rewording of the accept-audit long help's untouched sentences (acceptaudit.go:20-31).** No decision asks for it, and nothing forces it. No webstercli test limits the help's length: cli_test.go:60 checks only for stale batch-era wording. No `Pattern-Departure:` trailer covers it.
- **C5, re-wrapped comments from earlier steps (commit 9a9d55c9d, "splits long comment lines").** Forced by the diff's own new comments and the gate's comment lint. Not a finding.

**Per decision:**
- **I1:** built by C1, with C1a and C1b.
- **I2:** built by C3.
- **I3:** built by C2. The tolerance covers only a fixer commit the current gate closure saw pass (verifygate.go:395-404 uses the closure's `preFix`). Whether a restart between the failed and the passing evaluation leaves I3 unbuilt is the correctness lens's question, so I file nothing for it here.

**Findings**

1. **LOW / scope** — `internal/webstercli/acceptaudit.go:20-31`. The diff rewrites accept-audit help sentences that no goal point touches.
   - It drops the fact that "a repeated call is harmless".
   - It shortens the absent-contract `next` sentence and the `accepted` envelope sentence.
   - Evidence: diff piece 002, lines 18-31. Fix commit 2d94213d8's message says "shorten accept-audit help", but no test requires a shorter help (internal/webstercli/cli_test.go:60), and no trailer explains it.
   - Fix: restore the untouched sentences, the idempotency fact included, and keep only the added verify-gate and records-side clauses.

Tool calls:
- Grepped webstercli tests for a long-help length limit and showed commit 2d94213d8's stat, to see whether anything forced the help rewording. The manifest had neither.
