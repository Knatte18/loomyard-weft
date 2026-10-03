# Card 1 — verifytree: the shared verify function and the verified-tree record

**Create:**
- `internal/verifytree/doc.go`
- `internal/verifytree/verify.go`
- `internal/verifytree/marker.go`
- `internal/verifytree/testmain_test.go`
- `internal/verifytree/marker_test.go`
- `internal/verifytree/verify_integration_test.go`

**Edit:**
- `cmd/lyx/pathrules_test.go`

**Uses:**
- `internal/verifyrun/run.go`
- `internal/proc/proc_linux.go`
- `internal/lyxdirs/dirs.go`

**Intent:**
Add `internal/verifytree`, the one package function every plan-verify site calls (D7), plus the running marker the status hook reads (D14).
No site calls it yet; later cards move each site onto it.

`verify.go` declares:

- `Dir(anchorRoot string) string`: `<anchorRoot>/.lyx/verify`, joined from `lyxdirs.DotLyxDirName`, so the Lyxdirs Single-Declarer Invariant holds and each teller (hubgeom, standalonegeom, loomcli) calls it rather than joining its own.
- `Paths{Worktree, Record, Marker, Log string}` and `NewPaths(worktree, dir string) Paths`, which names `verified-tree.yaml`, `running.yaml` and `verify.log` inside `dir`.
- `Site{Label string; Attempt int}`: the site label (`webster gate`, `Webster-Burler gate`, `Publish`, `Finalize`, `webster verify`) and the attempt, 0 where none applies.
- `Status` with `StatusPassed`, `StatusSkipped`, `StatusFailed` and `StatusDirty`, and `Result{Status, Dirty []string, ExitCode int, Tree string, Detail string}`.
- `DirtyPaths(worktree string) ([]string, error)`: `git status --porcelain` with ignored files excluded, through `gitexec` (a read-only verb the Fabric Git Invariant exempts), returning the dirty paths.
- `Verify(ctx, p Paths, site Site, command string) (Result, error)`, in this order:
  1. a non-empty `DirtyPaths` returns `StatusDirty` with the paths and runs nothing;
  2. when the record holds HEAD's tree SHA (`git rev-parse HEAD^{tree}`) and the same command, it returns `StatusSkipped` without running;
  3. otherwise it writes the marker, runs the command through `verifyrun.Run` with output to `p.Log`, removes the marker whatever happened, and on exit 0 writes the record `{tree, command, verified_at}` and returns `StatusPassed`.
  A non-zero exit is `StatusFailed` with the exit code;
  a shell that could not start is `StatusFailed` with exit code -1 and the cause in `Detail`;
  a cancelled ctx is a returned error, and writes no record.

The record is written only here and only after a pass, so a crash anywhere before that write leaves a mismatch and the next call runs again.
The skip lets through only a tree byte-identical to one that passed the same command.
An unreadable or malformed record reads as no record, which only costs a re-run.

`marker.go` declares `Marker{Site string; Attempt int; Started time.Time; PID int}`, the marker `Verify` writes (its own pid), and `ReadMarker(path string) (Marker, bool, error)`, which reports false for an absent marker or one whose pid `proc.IsAlive` reports dead.
`Marker.Note()` renders `verify <site> (attempt N, since HH:MM)`, dropping the attempt clause when Attempt is 0.
Liveness goes through a package seam so `marker_test.go` stays tier 1.

Tests:
`marker_test.go` covers the note with and without an attempt, an absent marker, a dead-pid marker and a malformed one.
`verify_integration_test.go` (integration tag, real scratch repo) pins the decision record's shared-function test: a skip when the record names HEAD's tree and the command, a run when either differs, a dirty tree refused with its paths and no run, the record written only on a pass, an interrupted run (cancelled ctx) leaving no record so the next call runs, and the marker gone after both a pass and a failure.
`testmain_test.go` calls `gitkit.HermeticGitEnv()` and `tmuxkit.Main`, as `internal/verifyrun/testmain_test.go` and `internal/webstercli/testmain_test.go` do.

`cmd/lyx/pathrules_test.go` gains a `classTransient` row for `verifytree.Dir`.

**ImpactSummary:** new leaf package with no callers yet; the path-rules guard gains one transient row.
