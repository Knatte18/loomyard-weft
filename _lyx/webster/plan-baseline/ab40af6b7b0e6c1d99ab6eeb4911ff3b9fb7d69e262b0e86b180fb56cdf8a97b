# Card 5 — Webster scans the root module for guard tests

**Create:**
- `internal/websterengine#guardTests`
- `internal/websterengine#scanGuardTests`
- `internal/websterengine/guardstep_test.go`
- `internal/websterengine#writeGuardFixture`

**Uses:**
- `internal/impactset#FindGuardTests`
- `internal/impactset#GuardScanError`
- `internal/impactset#IsGuardScanError`

**Intent:**
Decision D2's input and D3's error policy: find the root module's `//lyx:guard` tests without spawning a process.
Both new members go in a new file `internal/websterengine/guardstep.go`, whose header comment says it derives the guard step every card gate ends with.

`guardTests` holds what one begin-batch or recover-batch render found:
`names`, a map from a worktree-relative guard package directory (`.` for the root) to its sorted guard test names, holding only directories with at least one;
`goFiles`, a map from each of those directories to the worktree-relative, slash-separated paths of the `.go` files it directly holds;
`scanErr`, the `*impactset.GuardScanError` the scan returned, nil when it accepted every test file.
The zero value renders no guard step.

`scanGuardTests(worktree)`:

- With no `go.mod` at the worktree root it returns the zero value and no error.
- Otherwise it walks the worktree with `filepath.WalkDir` and collects each directory that directly holds a `_test.go` file, recording every `.go` file it holds.
  It skips, below the root, a directory named `testdata` or `vendor`, a directory whose name begins with `_` or `.`, and a directory holding a `go.mod`, with its whole subtree, as `go list ./...` does, so no file under them is scanned.
- A walk error, a directory or entry it cannot read, is returned wrapped with the path it names: `webster: walk the worktree for guard tests: <path>: <error>`.
  The walk also opens and closes each `_test.go` file it collects, and a failed open is returned the same way, so an unreadable test file is D3's read error and never reaches `FindGuardTests`, whose parse failure would otherwise make it a `GuardScanError`; `impactset`'s classification, and so the round gate's error policy, stays unchanged.
  This is not a test-file fault, and a later card turns it into begin-batch's and recover-batch's refusal.
- It passes the collected directories to `impactset.FindGuardTests`.
  A `GuardScanError` (`impactset.IsGuardScanError`) is kept in `scanErr` with `names` empty, and no error is returned, so the render goes on without a guard step.
  Any other error is returned wrapped, as a read failure.

`writeGuardFixture`, in the new `internal/websterengine/guardstep_test.go` (package `websterengine`), writes each slash-separated path of `files` under `root` with its content, creating directories; `writeGateFile` writes only `x`, which no guard scan can parse.
Card 6's guard rows reuse it.

`TestScanGuardTests`, in the same file, is a table over fixture trees written under `t.TempDir()` with `writeGuardFixture`; each row asserts the exact `names`, the `goFiles` of each guard directory, whether `scanErr` is set (and its `File`) and the returned error:

- no root `go.mod`, with a marked test present: zero value, nil error;
- a root `go.mod` and no marked test: empty `names`, nil error;
- a root `go.mod`, marked tests in `internal/a` (two files, one non-test `.go` file beside them) and at the root: both directories with their sorted names and every `.go` file;
- marked tests under `testdata/x`, `vendor/x`, `_skip/x`, `.hidden/x` and `nested/x` beside a `nested/go.mod`, each also holding a misplaced marker so a scan would fail: none appears and `scanErr` stays nil;
- a misplaced `//lyx:guard` marker in `internal/a/a_test.go`: `scanErr` set with `File` `internal/a/a_test.go`, nil error;
- a root `go.mod` and an unreadable directory (mode `0o000`, restored in `t.Cleanup`; the row is skipped on Windows and as root, as `rebaseline_test.go` does): an error naming that directory's path, and no `scanErr`;
- a root `go.mod` and a `_test.go` file with mode `0o000`, restored and skipped the same way: an error naming that file's path, and no `scanErr`.

The package's `test-budget.yaml` row has room for this test beside card 2's.

**Commit:** 5: Webster scans the root module for guard tests
