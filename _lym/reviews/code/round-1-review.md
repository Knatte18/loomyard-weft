---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: consistency
    location: internal/gatecli/strayfiles_integration_test.go:26
    summary: TestRunTierStrayFiles skips itself on Windows at run time, which PATTERN-test-speed's No test skips by design rule forbids for an OS-bound test.
    fix: "Add the build constraint line `//go:build !windows` above the file-level comment of internal/gatecli/strayfiles_integration_test.go, separated from it by a blank line. Delete the `if runtime.GOOS == \"windows\" { t.Skip(...) }` block in TestRunTierStrayFiles and the now unused \"runtime\" import."
    origin: assist:assist
  - id: F2
    severity: MEDIUM
    class: design
    location: internal/gatecli/copies.go#addFailedRuns
    summary: On a --copies run the stray-file failure lives only on the merged result, so the failure report leaves it out, and no test covers a copies run that leaves a file.
    fix: "In internal/gatecli/copies.go addFailedRuns, after the loop over result.copies, add the stray-file cause from the merged result's causes to the report as a failed run of its own, labelled runLabel(run), when checkStrayFiles set it. A simple way is to keep the stray line on a new tierResult field (for example strayLine string) set by checkStrayFiles, and in addFailedRuns append a failedRun{label: runLabel(run), outcome: \"exit 1\", log: logPath, causes: []string{result.strayLine}} when that field is non-empty and result.copies is non-empty. Add a row to TestRunTierStrayFiles in internal/gatecli/strayfiles_integration_test.go that sets request.copiesRun to 2 with the command `touch c.lock`. Assert that the result has exit 1, that its causes hold the stray line exactly once, and that addFailedRuns on an execution yields a failed run whose causes hold the line."
    origin: lens:code-correctness
  - id: F3
    severity: MEDIUM
    class: design
    location: internal/gatecli/tierrun.go:75
    summary: Each retryTier attempt takes its own listing before it runs, so a file left by a first attempt that failed on a shortage or crashed counts as present before the rerun, and a passing rerun lets it through.
    fix: "Take the listing before the run once per tier run, outside retryTier. In internal/gatecli/run.go runGreenAware, take `before, listed := untrackedWorktreeFiles(e.request.root)` when run.invocations is non-empty, before calling retryTier. Pass before and listed into the runTier call, for example as new runTier parameters, and drop the listing from runTier itself. Keep runListed's empty-scope call free of any listing. Add a row to TestRunTierStrayFiles that calls runTier twice with the same before map: the first command is `touch r.lock; exit 1`, and the second is `true`. Assert that the second result has exit 1 and names r.lock."
    origin: lens:code-correctness
  - id: F4
    severity: LOW
    class: design
    location: internal/gatecli/tierrun.go:83
    summary: No test row covers the refusal skip; the mutation survivor at tierrun.go:83 names it.
    fix: "Add a row to TestRunTierStrayFiles with two invocations: the first in root with the command `touch r.lock`, the second in a directory under root that does not exist, so the shell start becomes a refusal. Assert that result.refusal is non-empty, that stderr holds no stray-file line, and that result.causes is empty. The table needs a field for the extra invocation, or a field naming the second invocation's dir."
    origin: lens:code-tests
  - id: F5
    severity: LOW
    class: design
    location: internal/gatecli/strayfiles.go:74
    summary: strayFilesLine joins raw file names, so a name holding a newline breaks the one-line output and a name holding a comma reads as two paths.
    fix: "In internal/gatecli/strayfiles.go strayFilesLine, format each named path with strconv.Quote before joining, keeping the map keys and comparison on the raw names. Update the expected prefix in TestRunTierStrayFiles to quoted names, and add a row whose command is `touch 'a\nb'` (a name with a newline) that asserts one stderr line naming \"a\\nb\" quoted."
    origin: assist:assist
  - id: F6
    severity: LOW
    class: design
    location: internal/gatecli/strayfiles_integration_test.go:51
    summary: The truncation row derives both its fixture and its expected names from strayFilesListed, the same slice the code takes, so the test cannot catch a change to the cap.
    fix: "In TestRunTierStrayFiles, replace the generated many slice with a literal fixture of the twelve names f00 through f11. Give the truncation row the literal wantNamed f00 through f09 and wantRest 2."
    origin: assist:assist
---

### [BLOCKING] The new test skips on Windows instead of being left out of the Windows build

**Location:** internal/gatecli/strayfiles_integration_test.go:26

**Issue:** `TestRunTierStrayFiles` calls `t.Skip("the tier command is a POSIX shell")` on Windows.
`pattern/PATTERN-test-speed.md`, under "No test skips by design", says a test that means something on one OS alone sits in a file only that OS compiles, through a `_linux` name or a `!windows` build constraint.
The PATTERN entry is a listed constraint, so the violation is BLOCKING.
Older tier0 tests in `tierrun_test.go` and `copies_test.go` use the same skip, but this file is new in the diff.

### [MEDIUM] A copies run drops the stray-file failure from the failure report

**Location:** internal/gatecli/copies.go#addFailedRuns

**Issue:** `checkStrayFiles` runs on the merged result `runTierCopies` returns.
It appends the line to the merged `causes` and sets the merged exit code to 1.
`report` (`run.go:722`) then calls `addFailedRuns`, which reads only `result.copies`: it skips every copy whose own exit code is 0 and takes each red copy's own `causes`.
When every copy passed, the call exits 1 and the failure report lists no failed run.
When a copy failed for some other reason, the stray line is still missing from the report.
That contradicts the doc's `# Failure report`, under which one read of the report shows every failure.
The stderr line and the merged log keep the line.
The diff's doc claim that a `--copies` run is listed once around all its copies has no test: the copies tests run in a root with no repository.
The code-correctness, code-affected-set and code-tests lenses and the assistant each found this.

### [MEDIUM] A rerun treats the first attempt's stray files as already present

**Location:** internal/gatecli/tierrun.go:75

**Issue:** `runGreenAware` (`run.go:628`) calls `runTier` inside `retryTier`, so each attempt takes a new listing before it runs.
`retryTier` reruns an attempt that failed on a resource shortage or crashed its runner.
A file such an attempt left is in the rerun's own first listing, so a passing rerun passes the tier and the file stays in the worktree.
The first attempt's line survives only in its run record, which `recordFirst` writes and no report reads.
The doc's `# Stray files` rule, that a file the second listing holds and the first lacks fails the run, then fails for the tier run as a whole.
The window is narrow (a rerun only follows a shortage or a crash), so MEDIUM rather than BLOCKING.

### [LOW] The refusal skip has no test row

**Location:** internal/gatecli/tierrun.go:83

**Issue:** `runTier` skips the check when the result carries a refusal.
Every row of `TestRunTierStrayFiles` has one invocation that starts, so no row reaches a refusal after a successful listing.
The existing refusal test runs outside a repository, so `!listed` short-circuits there.

### [LOW] File names are not escaped in the stray-file line

**Location:** internal/gatecli/strayfiles.go:74

**Issue:** `-z` keeps a name with a newline or comma intact, but `strings.Join(named, ", ")` writes it raw.
A newline splits the one line the doc promises, and a comma makes the list ambiguous for the "remove the files by their paths" way forward.

### [LOW] The truncation row's expectation repeats the code's slice

**Location:** internal/gatecli/strayfiles_integration_test.go:51

**Issue:** The fixture has `strayFilesListed+2` names and expects `many[:strayFilesListed]`, which is how `strayFilesLine` cuts the list.
A changed cap moves the test with it, so the row does not pin the cap at its stated value.

## Rejected

- Assistant F2 (existing tier0 tests now spawn git): `TestRunTierProcessCap`, `TestRunTierMemoryGuard`, `TestRunTierStartFailureKeepsTheShellRefusal`, `TestRunTierNoTestNotice` and `TestRunTierCopies` already spawn a shell per run at the base. The added `git ls-files` fails fast outside a repository and adds no new kind of dependency, and moving five unchanged tests to another tier is outside this task's scope.
- Assistant F5, beyond the refusal row kept as F4: the no-invocation skip cannot be observed, since a run with no invocation runs no command. A failed second listing needs a fault seam the package lacks. A cancelled or timed-out run keeps its own non-zero exit, and the check only adds a line.
- Code-tests NIT on the mutation survivors at `strayfiles.go:21` and `tierrun.go:74`, `:75` and `:83`: the tier1 rows of `TestRunTierStrayFiles` kill all of them but the refusal half of `:83`, so it is no finding. That remaining half is F4.
- Assistant F1 at BLOCKING: kept as F3 at MEDIUM. A rerun follows only a shortage or a crash, and the shortage marker in a file name is contrived.
