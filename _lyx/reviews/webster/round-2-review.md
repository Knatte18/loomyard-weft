---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: scope
    location: internal/fabricvocab/leaf_enforcement_test.go:1
    summary: internal/fabricvocab has tests but no TestMain calling tmuxkit.Main, so TestTmuxIsolation_EveryTestPackageRunsThroughTmuxkitMain fails
    fix: Add internal/fabricvocab/testmain_test.go in package fabricvocab, with a header comment like the sibling testmain_test.go files, containing func TestMain(m *testing.M) { os.Exit(tmuxkit.Main(m)) } and importing os, testing and github.com/Knatte18/loomyard/internal/testkit/tmuxkit. Then run lyx gate test ./cmd/lyx and lyx gate test ./internal/fabricvocab.
---

### [BLOCKING] internal/fabricvocab lacks a tmuxkit TestMain

**Location:** `internal/fabricvocab/leaf_enforcement_test.go:1`
**Issue:** The Publish verify failed on `TestTmuxIsolation_EveryTestPackageRunsThroughTmuxkitMain` (`.lyx/verify/verify-4.log`, lines 1–9), which reports `PATTERN-test-isolation` violated for `internal/fabricvocab` under every tag set.
The package arrived through the merge-in commit 35f6a6880 (introduced by 7931932b7); it holds `leaf_enforcement_test.go` and no `TestMain`.
This branch made every test package run through `tmuxkit.Main` (cards 7–11), so a package merged in from the parent must follow the same rule.
Other packages carry a `testmain_test.go` calling `os.Exit(tmuxkit.Main(m))` (for example `internal/agentname/testmain_test.go`).
The remaining packages in the log pass, so this is the only package missing it.

## Focus directive verdicts

- `cleanupHandoffs` in `internal/orchengine/cleanup.go`: not a defect.
  The lock is acquired when `apply` is true or the lock file exists, so a dry run with the file absent does not create it, and apply always takes the blocking `lock.AcquireWriteLock`.
  A dry run with an existing lock file still takes the lock and reads state safely.
  `TestCleanupHandoffs` adds the row "an absent state lock file is not created by a dry run", which deletes the lock file, runs the pass and asserts the file is still absent when `apply` is false.
- `CleanupReport.Applied` in `internal/battencli/cleanup.go`: not a defect.
  The comment now reads "ran with apply true, a dry run being false", and the verb sets `Applied: apply` (line 103) and prints `report.Applied` as `applied` (line 409).
