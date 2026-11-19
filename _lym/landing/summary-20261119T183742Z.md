# Fail a gate tier run that leaves new untracked files in the worktree

`lym gate test` now fails a tier run that leaves a new untracked, non-ignored file in the worktree.
That catches the whole class behind the stray zero-byte `internal/hardenshed/.lock` that stopped `hardener-campaign`'s Finalize.

## What changes

- `internal/gatecli/strayfiles.go` (new) lists the worktree's untracked, non-ignored files with `git ls-files --others --exclude-standard -z` through `gitexec.Run`.
  `checkStrayFiles` compares the listing taken after the run with the one taken before it.
  When the run left new files, it writes one line to stderr and the log before the tier's summary line and adds the line to the failure report's causes.
  A run that passed gets exit code 1; a run that failed keeps its own code.
  The line names the sorted new files, at most `strayFilesListed` of them and a count of the rest.
  It gives the way forward: make the test write under its temp directory, remove the files, then re-run.
- `runTier` in `internal/gatecli/tierrun.go` now wraps the check around the tier.
  The former body is now `runTierCommands`, which runs the tier once.
  A `--copies` run is listed once around all its copies, so `runTierCopies` calls `runTierCommands` for each copy.
  A run with no invocation takes no listing.
  When git cannot list the files, as in a directory outside every repository, the run is not checked; that is logged at info.
- `internal/gatecli/doc.go` adds a `# Stray files` section with the rule and its bound.

## Design decisions

- **No hardenshed test changed.**
  The board entry's first idea was to point the offending hardenshed test at `t.TempDir()`.
  Every hardenshed test builds its record paths under `t.TempDir()`.
  Their locks are named `<record>.json.lock` or `hubs.json.lock`.
  `lock.TrackedFileLockPath` gives a bare `.lock` only for an empty path, and no current test passes one.
  `lym gate test ./internal/hardenshed` on the base passed 93 of 93 tier0 tests and left the package directory clean.
  The decision record's `Added after Discussion` entry "No hardenshed test fix; stray-file gate check instead" drops the test fix.
  The run builds only the gate check.
- **Before-and-after difference rather than "no untracked files".**
  A worktree routinely holds untracked work in progress, so only files that appear during the run count.
  Ignored files never count, and neither do files the record and cache directories hold, since git ignores them.
- **Git CLI over go-git status.**
  `git ls-files` honours `.git/info/exclude` and stays fast on a large tree.
  The call goes through `gitexec.Run`, as `green.go`'s HEAD read does.

## Testing

`TestRunTierStrayFiles` in `internal/gatecli/strayfiles_integration_test.go` is tier1 because it needs a real git repository.
It drives `runTier` with a shell command in a fresh repository, one row per case:
- a new file fails a passing tier;
- a failing tier keeps its exit code;
- a file untracked before the run, an ignored file and a file the run removes again are all not named;
- the files beyond the cap are counted;
- a directory outside every repository is not checked.

## Merge danger

**Door:** two-way
**Blast radius:** gate

A false positive fails every gate tier run in that worktree.
That would happen for any existing test or tier command that leaves a non-ignored file in the tree, or for another process writing there while the tier runs.
The check names the files, so such a failure is easy to diagnose, and a revert removes it.
