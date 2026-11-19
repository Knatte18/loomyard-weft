```yaml
lens: code-scope
outcome: reported
```

Task: code-scope lens. I traced each change in the diff to the board entry's goal points.

**Decisions.** The record has a decision record only: `## Decisions` is empty and there are no `Added after Discussion` entries.
- **G1 (Goal, Idea 1):** find the writer by recording which packages ran when the entry appeared, or by watching the temp root for the create and naming the process, and fix that test.
- **G2 (Goal, Idea 2):** blame only the package that wrote the entry, or report a machine finding that does not fail a package whose own run did not create it.

**Per change:**
- **C1** `internal/testkit/tmuxkit/gitentry.go`: `watchGitEntries`, `snapshotProcesses`, `classifyProcesses`, `descendsFrom`, `quoteArgv` and `gitEntrySighting`. Traces to G1's instrument half (watch the create, name the processes) and feeds G2.
- **C2** `gitentry.go` `checkNewGitEntries` and `describeGitEntry`: the new blame rule. A package fails only when no other test binary ran; otherwise it gets a machine finding. Traces to G2.
- **C3** moves `gitEntryName`, `sharedTempDirs`, `presentGitEntries`, `runCheckingGitEntries` and `removeStrayGitEntry` from `tmuxkit.go` to `gitentry.go`, and drops the `io/fs` import. Forced by C1/C2: one file owns the concept. No finding.
- **C4** `tmuxkit.go`: the package doc paragraph and the `Main` call sites passing `snapshotProcesses`. Forced by C1/C2.
- **C5** `internal/fswatch/testmain_test.go` becomes `package fswatch_test`. Forced by C1: tmuxkit now imports fswatch, and an in-package test importing tmuxkit would be an import cycle.
- **C6** `tmuxkit_test.go` rows, `TestClassifyProcesses` and `tmuxkit_snapshot_linux_test.go`. Forced: these are the tests of C1/C2.
- **C7** `pattern/PATTERN-test-isolation.md` `.git` bullet. Forced: the rule document C2 made stale.

**Per decision:**
- **G1:** C1 builds the instrument. No change finds the writer or fixes a test.
- **G2:** C2 builds it, with C3–C7 forced alongside.

No change traces to no decision.

**Findings**

1. **BLOCKING · scope**
   - **Location:** the review range as a whole; nothing in the diff carries it.
   - **Summary:** G1's "and fix that test" is not built. No test or production path that writes `/tmp/.git` is found or fixed. `_lym/landing/summary.md` says so itself ("**The writer is not fixed in this change.**") and frames it as the writer's own design call. It is not named there as a parent correction, and no `Added after Discussion` entry licenses the deferral.
   - **Evidence:** `_lym/discussion/decision-record.md:19-20` (Idea 1: "…and fix that test"); `_lym/landing/summary.md` "Design decisions", first bullet; the diff pieces 001–003 touch no writing test.
   - **Fix:** find and fix the writing test in this range, or get the parent to record the deferral with `lym loom decision add`.
2. **MEDIUM · design**
   - **Location:** `internal/testkit/tmuxkit/gitentry.go:146-148` (`checkNewGitEntries`), and the documented bound at `internal/testkit/tmuxkit/tmuxkit.go` package doc and `pattern/PATTERN-test-isolation.md:24`.
   - **Summary:** G2 asks that a package whose own run did not create the entry not be failed. The rule built is narrower: it fails any package that ran with no other `*.test` binary alive. So it still fails a package for an entry written by a non-test process. Examples: a daemon a finished test of another package left behind, a `go build` or `go vet`, or any other process on the machine. The diff documents this as a bound, but it contradicts G2's either/or. The entry's machine-finding branch would have satisfied G2 for every unattributable case.
   - **Evidence:**
     - `_lym/discussion/decision-record.md:21`
     - `gitentry.go:146` (`if seen && sighting.attributed && len(sighting.others) == 0 { code = 1 }`)
     - `gitentry.go:227`, where `others` counts only `.test` binaries
     - `pattern/PATTERN-test-isolation.md:24`, the diff's own bound text
   - **Fix:** fail only when the snapshot ties the entry to this binary or its descendants, for example a descendant working in `dir`. Otherwise report a machine finding, or get the parent to record the narrower rule.

Tool calls: none. The context already held the decision record, the summary, the commit log and every diff piece.
