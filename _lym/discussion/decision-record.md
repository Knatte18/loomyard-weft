# Decision record — No test is skipped by design

## Goal

A gate log's skips are only missing prerequisites, so every skip it shows is worth reading.
No test in the suite skips itself because of the OS it runs on, because its body is disabled, or because it is a child-process body.

## Scope

- Every test or subtest that calls `t.Skip` on a `runtime.GOOS` check alone.
- Tests that skip on a combined check, such as `runtime.GOOS == "windows" || os.Geteuid() == 0`: the OS half goes, the root half stays.
- Tests whose skip stands for an OS-only capability, such as `proc.StartTime` being unreadable, which happens only on Windows.
- `TestBuildLoomShed_OutputShape` in `internal/loomcli/sharedbootstrap_test.go`.
- The subtest in `internal/loomshed/verifygate_integration_test.go` that skips unconditionally.
- The omitted cells of `TestCrossProduct` in `internal/fabricengine/livestate_matrix_test.go`.
- The child-process bodies `TestPushCoalescedChildProcess` and `TestLockHolderChildProcess` in `internal/gitrepo/push_test.go`.
- Out of scope: skips for a missing prerequisite, such as an absent binary, no symlink privilege, running as root, a missing .NET SDK or NuGet packages, and `TestClaude/neither_skips/inner` in `internal/testkit/llmkit`.

## Decisions

### D1 — A whole test meant for one OS lives in a file constrained to it

A test function that skips itself on every OS but one, or on one OS alone, moves into a file only that OS compiles: a `<base>_windows_test.go` or `<base>_linux_test.go` name, or a `<base>_unix_test.go` file whose build line adds `&& !windows` to its tier tag.
The new file keeps the tier tag of the file it came from, and the OS check and its `t.Skip` go.
A remaining prerequisite check in the moved test, such as running as root, stays.
Rationale: the board entry's first rule; a file name or build line is the repo's existing idiom for OS-only code.

### D2 — A table row meant for one OS is not generated elsewhere

A table-driven test whose rows carry an OS flag, or whose subtest skips on `runtime.GOOS`, filters those rows out before `t.Run` instead of running them into a skip.
When most rows of a table are OS-bound, the rows move into the OS-constrained file of D1 instead.
Rationale: one row of a shared table cannot sit behind a build tag without splitting the table; not generating the row matches the matrix decision D4.

### D3 — `TestBuildLoomShed_OutputShape` is deleted

The test's receiver has no fabric behind it, so `buildLoomShed` always fails and the test always skips.
Rationale: building a real fabric would move the test to `tier1` to pin a path pass-through and a producer count that every loom step run in the integration tiers already reaches.

### D4 — An omitted matrix cell is not generated

`TestCrossProduct` generates no subtest for a verb and state pair the `Omissions` table lists, and its cell tally counts the omitted pairs from the table instead of from skipped subtests.
Rationale: the table already records each omission and its reason, so the skipped subtest added nothing but a skip line.

### D5 — The child-process bodies run from `TestMain`

The two child bodies stop being test functions.
`push_test.go` registers each body by its environment variable, and the package's `TestMain` runs the body whose variable is set and exits before `m.Run`.
The parent tests spawn the test binary with the variable set, as now.
Bound: a body runs only when its variable is set, which only the parent test's spawn does.

### D6 — The unconditionally skipped verify-gate subtest is deleted

The subtest `fails with the file and line of a misplaced marker` skips before its body because the gate key carries no `--base`.
A board note records the coverage to restore once the key reads its diff base.
Rationale: a test that always skips is a test meant to skip.

## Constraints

- `PATTERN-test-speed`: every new test file carries exactly one tier tag, a platform term beside it; tier purity, the llm-tier and tmux-tier scans in `cmd/lym` stay green.
- `PATTERN-test-isolation`: the gitrepo `TestMain` keeps `gitkit.HermeticGitEnv` and `tmuxkit.Main`.
- Moved tests keep their names, so a gate log or timing report still finds them.

## Auto-mode assumptions

- A skip on "running as root" is a missing prerequisite, a non-root user, and stays.
- A skip on an absent loose object in `warpbinding_clone_integration_test.go` is an environment condition and stays.
- `proc.StartTime` is unreadable only on Windows, so a test needing it is Linux-only.
- Only Linux and Windows are supported hosts, so a test gated on `runtime.GOOS != "linux"` is a Linux-only test.

## Open risks

- A Windows-only file is compiled only on Windows, so this Linux run cannot build it; a mistake in a moved Windows test shows only on a Windows gate.
  `go vet` with `GOOS=windows` over the touched packages is the check.

## Acceptance criteria

- No test outside the scope's out-of-scope list calls `t.Skip` on a `runtime.GOOS` check (D1, D2).
- Each moved test keeps its name and tier, and compiles only on its OS (D1).
- OS-bound table rows are absent from a run on another OS, not skipped (D2).
- `TestBuildLoomShed_OutputShape` no longer exists (D3).
- `TestCrossProduct` reports no skipped subtest, and its tally still fails when the ran count disagrees with the `Verbs`, `States` and `Omissions` tables (D4).
- `TestPushCoalesced`'s cross-process and crashed-holder steps still pass, and no test named `TestPushCoalescedChildProcess` or `TestLockHolderChildProcess` exists (D5).
- The always-skipped verify-gate subtest is gone and a board note names the coverage to restore (D6).
- `lym gate test` is green over the diff on Linux, and `GOOS=windows go vet -tags <tier>` compiles the touched packages.
