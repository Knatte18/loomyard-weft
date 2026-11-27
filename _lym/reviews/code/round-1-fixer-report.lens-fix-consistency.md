```yaml
lens: fix-consistency
outcome: reported
```

Task: fix-consistency check of the fixer's change set against its surroundings.

- **MEDIUM**: `internal/testkit/tmuxrec/session.go`, `PlaySession`'s `t.Skipf` on a GOOS mismatch (F3).
  - The skip contradicts `PATTERN-test-speed`, "No test skips by design": "A test that means something on one OS alone sits in a file only that OS compiles", "A table row or subtest bound to one OS is not generated on another", and "A skip that remains names a prerequisite the host lacks, such as a binary, a privilege or a non-root user".
  - On Windows and macOS the skip fires by design in `TestRecordings_Replay`'s subtests and in the PR-Review tests of `landingshed`.
  - It also leaves `PATTERN-test-stand-ins`' "The production code that reads the outside thing runs over every recording in a `tier0` test" unmet on those hosts.
  - The fix commit carries `Loosened-Check` lines and no `Pattern-Departure` line.
  - A counter-reading: a recording made on the host's OS is a prerequisite the host lacks. The fixer should decide this explicitly and record it in the fixer-report.
  - A `linux` build constraint is not a cheap alternative: `newPRReviewFixture` (`prreview_test.go`) builds the session world, and `finalize_pr_test.go` shares that file's fixtures.
- **LOW**: `_lym/landing/summary.md`, "Design decisions", the replay bullet (F4 twin left unchanged).
  - It still says a replay "answers a call that changes nothing on the server from the latest recorded answer to it".
  - The code now looks back only as far as the last change the replay passed (the `answer` loop in `tmuxrec.go` stops at the first call that is not a read), and fails otherwise.

I found no stale reference to `resolveShellPath`, the old panic text or the moved tests under `internal/`, `docs/`, `pattern/` or `cmd/`.

Tool calls:
1. `git grep` with an exclude pathspec over the repository: it failed on the pathspec syntax. My context lacked a repository-wide reference search.
2. `git grep` over `internal`, `docs`, `pattern` and `cmd` for `resolveShellPath`, the kit's test names and the panic text: my context had only the `internal` hits from before the rename.
