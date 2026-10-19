# Card 1 — lyxtemp-short-run-names

**Edit:**
- `internal/lyxtemp#Root`
- `internal/lyxtemp#openRoot`
- `internal/lyxtemp#Root.newRun`
- `internal/lyxtemp#TestNewRun_HoldsItsLockUntilRemoved`
- `internal/testkit/tmuxkit#TestCheckSocketPath`
- `internal/lyxtemp/doc.go`
- `pattern/PATTERN-lyx-temp-root.md`

**Create:**
- `plan:internal/lyxtemp#randomRunName` -> `func randomRunName() string`
- `plan:internal/lyxtemp#runNameAttempts` -> `const runNameAttempts = 16`
- `plan:internal/lyxtemp#TestNewRun_RetriesATakenNameUpToTheBound` -> `func TestNewRun_RetriesATakenNameUpToTheBound(t *testing.T)`

**Uses:**
- `internal/lyxtemp#runPrefix`
- `internal/lyxtemp#newTestRoot`
- `internal/testkit/tmuxkit#checkSocketPath`

**Intent:** D1: a run directory is named `r` plus six random hex digits, so the longest tmux socket path under a 10-digit uid stays under the 107-byte `sun_path` limit (`/tmp/lyx-4294967295/r0a1b2c/tmux-4294967295/` plus the 61-byte key is 105 bytes).
This card lands first, since no test package runs on the Ubuntu host before it.

- `randomRunName` returns `runPrefix` followed by six lower-case hex digits drawn from `math/rand/v2` (no new import outside the standard library, so `TestLeafInvariant_AllowlistOnly` keeps passing).
- `Root` gains an unexported field `runName func() string`, the name source, replaceable so a test forces a taken name; `openRoot` sets it to `randomRunName`.
- `Root.newRun` replaces `os.MkdirTemp(r.path, runPrefix)` with a loop of at most `runNameAttempts` tries: each draws `r.runName()` and calls `os.Mkdir(filepath.Join(r.path, name), 0o700)`; an `fs.ErrExist` failure draws a fresh name, any other error returns at once wrapped as today (`create run directory: %w`).
  When every attempt hit a taken name it returns an error naming the root and the attempt count, for example `create run directory under %q: %d names taken`.
  The sweep, the lock and the inherited-root rule are unchanged: the `r` prefix stays, so `Root.Sweep`, `resolveRootPath` and run directories left by older binaries keep working.
- `TestNewRun_HoldsItsLockUntilRemoved` additionally asserts the created directory's base name matches `^r[0-9a-f]{6}$` on every platform, and that its mode is 0700 only off Windows, which reports no Unix permission bits, as `root_test.go` does for the root.
- `TestNewRun_RetriesATakenNameUpToTheBound` is a table over a `newTestRoot` root whose `runName` is replaced by a scripted sequence, with a directory pre-created under the root for the taken name:
  row "a taken name is retried" scripts the taken name then a fresh one and expects the fresh directory created and locked;
  row "the bound spent fails naming the root" scripts the taken name on every call and expects an error containing the root's path, exactly `runNameAttempts` draws, and no new entry under the root;
  row "another mkdir error returns at once" removes the root directory after `openRoot`, so `os.Mkdir` fails with `fs.ErrNotExist`, and expects exactly one draw and an error that `errors.Is` reads as `fs.ErrNotExist` and that is not the bound error.
- `TestCheckSocketPath` gains the row "run directory of the widest uid": dir `/tmp/lyx-4294967295/r0a1b2c`, uid 4294967295, `wantErr` false.
- `internal/lyxtemp/doc.go` and `pattern/PATTERN-lyx-temp-root.md` replace `os.MkdirTemp(root, "r")` with the new name: `r` plus six random hex digits, created 0700 with `os.Mkdir`, a taken name retried up to a bound and then refused naming the root.
  The PATTERN.md entry line itself is unchanged.

**ImpactSummary:** Changes only how lyxtemp names a run directory; every caller of `Dir`, `MkdirTemp` and `CreateTemp` sees a shorter path, and the sweep, lock and prefix rule are untouched.
