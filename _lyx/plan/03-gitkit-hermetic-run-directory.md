# Card 3 — gitkit-hermetic-run-directory

**Edit:**
- `internal/gitkit#HermeticGitEnv`
- `internal/gitkit#buildRepoTemplate`
- `internal/gitkit#allowedImports`
- `internal/gitkit/doc.go`
- `pattern/PATTERN-leaf-packages.md`
- `cmd/lyx/testdata/test-budget.yaml`

**Create:**
- `internal/gitkit/rundir_integration_test.go`

**Uses:**
- `internal/lyxtemp#CreateTemp`
- `internal/lyxtemp#MkdirTemp`
- `internal/lyxtemp#ProcessRoot`
- `internal/testkit/tmuxkit#Main`

**Intent:**
`gitkit.HermeticGitEnv` writes its neutral gitconfig through `lyxtemp.CreateTemp("gitkit-gitconfig-*")` instead of `os.CreateTemp("", ...)`, so the file lives in the process run directory and goes with it when `tmuxkit.Main` removes that directory (D4).
This closes the live leak: `HermeticGitEnv` runs first in `TestMain`, before `tmuxkit.Main` redirects `TMPDIR`, so today its file lands in the bare system temp directory and is never removed.
`TestMain` still calls it first; it creates the run directory on first use, and `tmuxkit.Main` then gets the same one from `lyxtemp.Dir()`.
A `lyxtemp` error panics, as the function's other setup failures already do.
`buildRepoTemplate` creates its once-per-binary template through `lyxtemp.MkdirTemp("gitkit-repo-*")`.

`allowedImports` in `leaf_enforcement_test.go` gains `github.com/Knatte18/loomyard/internal/lyxtemp`, and the file's header comment names it;
`pattern/PATTERN-leaf-packages.md`'s `internal/gitkit` section adds `internal/lyxtemp` to the closed set, and `gofrs/flock` through it.
`internal/gitkit/doc.go` says the neutral gitconfig lives in the process run directory.

`rundir_integration_test.go` (`//go:build integration`) holds one top-level test that re-executes the gitkit test binary (`os.Args[0]`) as its own helper, with only flags (`-test.run=^<the test>$`), so `refuseCLIReexec` lets it through.
In helper mode, chosen by an environment variable the parent sets, the test writes `GIT_CONFIG_GLOBAL`, `os.TempDir()` and one `t.TempDir()` path to a file the parent names, then fails when the mode says fail.
The parent runs a passing and a failing helper and asserts, for each: the exit code is 0 and non-zero respectively;
every recorded path lies inside a run directory directly under `lyxtemp.ProcessRoot().Path()` other than the parent's own `lyxtemp.Dir()`;
and once the helper has exited, none of the recorded paths and not the helper's run directory exist.
That pins "a test binary under `tmuxkit.Main` leaves nothing outside the root after a passing run and after a failing test, and its gitconfig is gone after the run" on every platform the tier runs on; the bare-creation scan in card 12 pins the rest of the tree.
`cmd/lyx/testdata/test-budget.yaml`'s `internal/gitkit` row grows by one.

**ImpactSummary:** HermeticGitEnv's gitconfig and gitkit's repo template move into the process run directory for every test binary; no caller changes.
