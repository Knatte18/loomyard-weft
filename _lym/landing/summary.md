# No test is skipped by design

A gate log's skips now name only a missing prerequisite, such as an absent binary, a symlink privilege or a non-root user.
On Linux, tier0 reports one skip, the llmkit skip helper's own test `TestClaude/neither_skips/inner`, and tier1 and tier2 report none.

## What changed

- **OS-bound tests compile only on their OS.**
  A test that skipped itself on every OS but one moved into a file only that OS compiles: a `_windows_test.go` or `_linux_test.go` name, or a `<base>_unix_test.go` whose build line adds `&& !windows` to its tier tag.
  A file whose every test was OS-bound kept its name and gained the OS term.
  Moved tests keep their names and tiers.
- **OS-bound rows and subtests are not generated elsewhere.**
  A table row carries a flag, such as `posixOnly`, `modeBits` or `permissionBits`, and the loop passes over it with `continue` before `t.Run`.
  For a positional table of functions, a `posixOnly(run)` or `linuxOnly(run)` wrapper returns nil on the other OS, and the loop passes over a nil run.
  A standalone subtest is wrapped in `if runtime.GOOS ...`, or its chained `if !t.Run(...)` gains the OS condition.
- **Windows-or-root skips keep only the root half.**
  Helpers that skipped on both conditions now skip only for root: `skipAsRoot` in `websterengine`, `fabricengine` and `impactset`.
  Their callers are rows or subtests that Windows never generates.
- **`TestBuildLoomShed_OutputShape` is deleted.**
  Its receiver had no fabric behind it, so `buildLoomShed` always failed and the test always skipped.
- **The always-skipped verify-gate subtest is deleted.**
  `fails with the file and line of a misplaced marker` in `internal/loomshed/verifygate_integration_test.go` skipped before its body, because the gate key's call carries no `--base`.
  Board note `verifygate-misplaced-guard-test` records the case to add back once the key reads its diff base.
- **`TestCrossProduct` generates no omitted cell.**
  `isOmitted` replaces `omittedReason`, and `assertCellTally` checks the omitted count against `len(Omissions) * len(anchors)`, so an `Omissions` row naming no cell still fails the run.
- **The gitrepo child-process bodies are no longer test functions.**
  `push_test.go` registers `pushCoalescedChild` and `lockHolderChild` in `childBodies` from `init`.
  `TestMain` in `testmain_test.go` runs the body whose variable is set and exits before `m.Run`.
  The parent spawns the binary with `-test.run=^$`.
- `pattern/PATTERN-test-speed.md` gains a "No test skips by design" section stating the rule, and the `PATTERN.md` entry for `PATTERN-test-speed` gains a clause.
  `cmd/lym/tierpurity_test.go`'s allowlist entry follows the socket-file test to `internal/reedengine/socketfile_unix_test.go`.

## Design decisions

- Subtests inside a larger test are gated where `t.Run` is called rather than moved to their own files, the same as table rows (D2).
  Moving them would split a shared fixture.
- A test that needs `proc.StartTime`, which is readable only on Linux, counts as Linux-only: `TestLoopLockOwner` moved to `landingdeps_linux_test.go`, where it now fails instead of skipping if no start time is readable.
  `TestCheckAlive`'s `needsTicks` rows run only on Linux.
- `internal/shedverbs/loop_integration_test.go` keeps its tier1 tag because `TestMain` needs its helper roles on every OS.
  Only `TestLoopIntegration_ChildTreeDiesWithLoop` moved to `loop_unix_integration_test.go`.

## To check by hand

- Linux `go vet` passed under `tier0` through `tier3` on Linux for every touched package.
  With `GOOS=windows`, `go vet` under `tier0` through `tier2` showed no new error in the touched packages.
  It still reports the existing `undefined: mustTmuxOutput` in `internal/reedengine/contract_integration_test.go`, because that helper lives in the `tier2 && linux` file `switch_integration_test.go`.
  That break predates this change and is left alone.
- Windows-only files, such as `gate_lingering_test.go` and `smoke_lifecycle_windows_test.go`, were type-checked by that vet but not run.
