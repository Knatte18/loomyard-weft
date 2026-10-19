# Case brief: Plan-Bouncer's commit stages an empty `_lyx/webster` pathspec

## 1. The question the next judge must answer

Should `planCommitPathspec` stop adding `_lyx/webster` to the plan commit's pathspec when the only evidence for it is an archived webster run under `_lyx/plan/archive-*/webster`, while the live `_lyx/webster` directory is empty and untracked?
If so, the function's condition needs a code fix, which is outside the tackler's two-file write scope.

## 2. The facts

- The stop is `kind: producer`, `transient` empty, failed at `Plan-Bouncer`, history length 28.
  The error is `git commit ... -- _lyx/plan _lyx/webster: exit 1: error: pathspec '_lyx/webster' did not match any file(s) known to git`.
  Source: `_lyx/shed/ubuntu-2404-support/drive-reports/drive-envelope-20261019-194836-4e6b.json`, field `error`; `.lyx/shed/ubuntu-2404-support/steps/dbebb90e88637a05.trace.log`, line `shed: step refused` at 19:48:36.190.
- The pathspec is built by `planCommitPathspec` in `internal/loomcli/wiring.go:154-161`.
  Line 157 appends `websterengine.DirRel()` when `holdsFile(websterengine.Dir(anchor))` or any `ArchivedPlanWebsterDirs` entry satisfies `holdsFile`.
  The `||` lets the archive alone add the live path.
- The live directory `_lyx/webster` (weft repo, `ubuntu-2404-support-weft/_lyx/webster`) is empty: `ls -la` shows only `.` and `..`.
  `git -C ubuntu-2404-support-weft ls-files -- _lyx/webster` prints nothing, so git has no entry there, and an empty directory is never matched by a pathspec.
- The archive does hold files: `_lyx/plan/archive-20261019T172536Z/webster/` contains `outcome.yaml`, `summary.md`, `state.json` and a `plan-baseline/` set.
  The archive sits under `_lyx/plan`, which is already the first pathspec entry, so it is staged without the live `_lyx/webster` path.
- The doc comment at `internal/loomcli/wiring.go:146-153` says the webster directory "is tracked", and that the path is "left out otherwise, since git refuses a pathspec that matches nothing". The archive branch contradicts that: it adds the path when nothing under it is tracked.
- The weft repo's `git status` shows the plan files as modified (`M _lyx/plan/...`), so the plan commit itself has content to commit; only the webster pathspec fails.
- The driver's stop report says `no rule settles the stop: the tackler decides`.
  Source: `_lyx/shed/ubuntu-2404-support/drive-reports/drive-report-20261019-194836-0a4c.md`.

## 3. What I ruled out, and how

- A plain re-launch or a `goto` to `Plan-Bouncer`: the same pathspec is built from the same disk state, so the commit fails the same way. A resume would only settle it if the live `_lyx/webster` gained a tracked file, which no one has put there, and I may not change the run's files.
- An environment or transient fault: `transient` is empty, the failure is git's own refusal, and it reproduces from the function's logic.
- A missing file in the plan directory: `_lyx/plan` is present and holds the staged plan files.
- A `_lyx` path problem: `_lyx` is a symlink to `ubuntu-2404-support-weft/_lyx`, and the weft `ls-files` query resolves through it correctly, so the pathspec error is not a symlink artifact.
- A `resume` or `hand-back`: the fix is a condition in Go code that someone must read and judge, so this is not a run-parent scope choice.
- Attempts 1 and 2 concern other causes (a burler git-mutation guard and a tmux version floor), and their verdicts do not bear on this stop.

## 4. Hypothesis (not verified by running the code)

The intended condition is "webster's live directory is tracked, or its archived copy must be staged with the deletions of the live one".
The `||` with the archive clause is wrong: when the live directory is empty and untracked, the archive should not add `_lyx/webster`.
A likely fix is to drop the `slices.ContainsFunc(archived, holdsFile)` clause, or to test whether the live path is in the index (for example via `git ls-files`) before adding it, and to correct the comment at `wiring.go:146-153` to match.
The next judge should confirm this by reading the git rule for empty pathspecs, and by checking whether any run has relied on the archive clause to stage a deletion of a tracked live `_lyx/webster`.
