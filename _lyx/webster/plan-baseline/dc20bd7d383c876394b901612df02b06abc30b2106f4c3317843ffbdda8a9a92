# Card 1 — lyxtemp-package

**Create:**
- `internal/lyxtemp/doc.go`
- `internal/lyxtemp/lyxtemp.go`
- `internal/lyxtemp/root.go`
- `internal/lyxtemp/root_other.go`
- `internal/lyxtemp/root_windows.go`
- `internal/lyxtemp/run.go`
- `internal/lyxtemp/sweep.go`
- `plan:internal/lyxtemp#Root` -> `type Root struct`
- `plan:internal/lyxtemp#Root.Path` -> `func (r *Root) Path() string`
- `plan:internal/lyxtemp#ProcessRoot` -> `func ProcessRoot() (*Root, error)`
- `plan:internal/lyxtemp#openRoot` -> `func openRoot(path string, uid int, now func() time.Time) (*Root, error)`
- `plan:internal/lyxtemp#resolveRootPath` -> `func resolveRootPath(tempDir string, uid int) string`
- `plan:internal/lyxtemp#checkRoot` -> `func checkRoot(path string, uid int) error`
- `plan:internal/lyxtemp#rootName` -> `func rootName(uid int) string`
- `plan:internal/lyxtemp#runPrefix` -> `const runPrefix = "r"`
- `plan:internal/lyxtemp#lockFileName` -> `const lockFileName = "run.lock"`
- `plan:internal/lyxtemp#creationGrace` -> `const creationGrace = 10 * time.Minute`
- `plan:internal/lyxtemp#run` -> `type run struct`
- `plan:internal/lyxtemp#Root.newRun` -> `func (r *Root) newRun() (*run, error)`
- `plan:internal/lyxtemp#run.remove` -> `func (u *run) remove() error`
- `plan:internal/lyxtemp#SweepReport` -> `type SweepReport struct`
- `plan:internal/lyxtemp#SweepFailure` -> `type SweepFailure struct`
- `plan:internal/lyxtemp#Root.Sweep` -> `func (r *Root) Sweep(apply bool) SweepReport`
- `plan:internal/lyxtemp#processMu` -> `var processMu sync.Mutex`
- `plan:internal/lyxtemp#processRun` -> `var processRun *run`
- `plan:internal/lyxtemp#Dir` -> `func Dir() (string, error)`
- `plan:internal/lyxtemp#MkdirTemp` -> `func MkdirTemp(pattern string) (string, error)`
- `plan:internal/lyxtemp#CreateTemp` -> `func CreateTemp(pattern string) (*os.File, error)`
- `plan:internal/lyxtemp#Remove` -> `func Remove() error`
- `internal/lyxtemp/root_test.go`
- `internal/lyxtemp/sweep_test.go`
- `internal/lyxtemp/run_test.go`
- `internal/lyxtemp/leaf_enforcement_test.go`
- `internal/lyxtemp/testmain_test.go`

**Edit:**
- `pattern/PATTERN-leaf-packages.md`
- `docs/overview.md`
- `cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `internal/lock#AcquireWriteLock`
- `internal/lock#TryAcquireWriteLock`

**Intent:**
Add `internal/lyxtemp`, the leaf package that owns lyx's per-user temp root in the system temp directory and one locked run directory per process (D1, D2, D3).
It imports the standard library and `internal/lock` only.

Root (D1):
`rootName(uid)` is `lyx-<uid>` off Windows and `lyx` on Windows, where the system temp directory is already per user.
`resolveRootPath(tempDir, uid)` returns `filepath.Join(tempDir, rootName(uid))`, except when `tempDir` is itself a run directory under a lyx root (its base carries `runPrefix` and its parent's base is `rootName(uid)`): then it returns that parent, so a child that inherited a run directory as `TMPDIR` works beside its parent, never inside it (D3, moving `tmuxkit`'s `kitBase` detection here).
`openRoot` creates the root with `os.Mkdir(path, 0o700)` when absent, then runs `checkRoot`, whatever created it.
`checkRoot` in `root_other.go` (`//go:build !windows`) requires an `os.Lstat` directory with neither `os.ModeSymlink` nor `os.ModeIrregular`, permission bits exactly `0o700`, and a `syscall.Stat_t` owner equal to `uid`;
`root_windows.go` applies the real-directory check alone and ignores `uid`.
Every failure is an error naming the path, and the caller fails: nothing falls back to the bare system temp directory, and an error path creates nothing.
`ProcessRoot()` is `openRoot(resolveRootPath(os.TempDir(), os.Getuid()), os.Getuid(), time.Now)`.

Run directory (D1):
`Root.newRun` sweeps the root first (`Sweep(true)`, its failures dropped, since the next start retries them), then creates `os.MkdirTemp(root, runPrefix)` and takes a blocking `lock.AcquireWriteLock` on `lockFileName` inside it, so a sweeper's momentary hold delays the owner rather than failing it.
`run.remove` releases the lock first, then removes the directory, since Windows cannot remove a locked file.
The names stay short for `tmuxkit`'s socket limit: `/tmp/lyx-1000/r<10 digits>/tmux-1000/` plus reed's 61-byte key is 97 bytes, under `checkSocketPath`'s 107.

Sweep (D2):
`Root.Sweep(apply)` reads only the root's direct children; it considers an entry only when its name carries `runPrefix` and `os.Lstat` reports a directory that is no link.
Within `creationGrace` of its modification time, against the root's injected clock, an entry is skipped untouched.
Past it, an entry with no lock file is a candidate; an entry with one is a candidate only when `lock.TryAcquireWriteLock` takes it, and the sweep releases that lock before removing (Windows).
The sweep never creates a lock file: it stats the lock path first and calls `TryAcquireWriteLock` only on an existing file.
With `apply` true each candidate is removed; with it false nothing is removed and the report lists the same candidates, computed by the same code.
An entry that vanishes under a concurrent sweeper (a not-exist error at any step) is no failure; an entry it cannot lock or remove is a `SweepFailure` naming the path and the error, and is left.
Bound: it deletes only a prefixed run directory whose owner holds no lock and that is past the grace; it never reads inside a live run directory, never touches an unprefixed entry, a file or a link, and the grace covers the window between `mkdir` and the owner's lock.

Process API:
`Dir()` returns the process run directory, creating it through `ProcessRoot().newRun()` on first use under `processMu`; later calls return the same path.
`MkdirTemp(pattern)` and `CreateTemp(pattern)` are `os.MkdirTemp`/`os.CreateTemp` under `Dir()`.
`Remove()` removes the process run directory through `run.remove` and clears `processRun`, so a later `Dir()` makes a fresh one; it returns nil when none was created.

Tests, all untagged, in-package and parallel, each over its own root under `t.TempDir()` with an injected clock:
`root_test.go` covers `openRoot` creating a 0700 root, refusing a root that is a link on every platform, and off Windows refusing a mode other than 0700 and an owner other than the passed uid (`os.Getuid()+1`), each error naming the path and creating nothing beside or inside it; it also covers `resolveRootPath` for a plain temp directory, an inherited run directory under a root, and a run-prefixed directory whose parent is not a root.
`sweep_test.go` is one table over the acceptance rows: a dead owner past the grace and a lock-less prefixed directory past the grace removed; a live owner (the test holds its lock) kept at any age; a dead owner within the grace kept; an unprefixed entry, a prefixed file and a prefixed link kept, the link's target untouched; a dry run removing nothing and listing exactly what the apply row removes; two concurrent sweeps of one root both reporting no failure.
The dead-owner rows stand for a killed test binary, since the OS releases a dead holder's lock, which is how the next start's sweep removes a killed binary's run directory.
`run_test.go` covers `newRun` holding its lock (a `TryAcquireWriteLock` from the test fails), `remove` deleting the directory, a child run made through an inherited run directory landing beside the parent's while its sweep leaves the parent's directory and contents alone, and `Dir`/`MkdirTemp`/`CreateTemp` returning paths under one run directory without calling `Remove`, which the test binary's own run directory needs.
`leaf_enforcement_test.go` pins the import set with `scankit.AssertImportAllowlist(t, "internal/lyxtemp", "github.com/Knatte18/loomyard/internal/lock")`.
`testmain_test.go` is `package lyxtemp_test` calling `os.Exit(tmuxkit.Main(m))`; the external package avoids the cycle the next card makes when `tmuxkit` imports this package.

Docs in this commit: `doc.go` states the root, the run directory, the sweep and its bound;
`pattern/PATTERN-leaf-packages.md` gains an `internal/lyxtemp` section: imports only the standard library and `internal/lock`, and `gofrs/flock` through it, and is importable from production code, not a kit;
`docs/overview.md`'s package tree gains an `internal/lyxtemp/` row.
`cmd/lyx/testdata/test-budget.yaml` gains an `internal/lyxtemp` row equal to the package's top-level test count.

**ImpactSummary:** New leaf package and its docs; nothing calls it yet, so no existing behavior changes.
