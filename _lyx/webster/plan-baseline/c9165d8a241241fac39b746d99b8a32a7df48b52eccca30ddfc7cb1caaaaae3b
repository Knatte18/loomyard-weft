# Card 7 — fix-commit-check

**Create:**
- `internal/websterengine/integrationfix.go`
- `internal/websterengine/integrationfix_test.go`
- `plan:internal/websterengine#checkFixCommits` -> `func checkFixCommits(worktree, preFixHead, reportHead, planDir string, parentBranch ParentBranchFunc) (warning string, err error)`

**Edit:**
- `internal/websterengine/gitwrap.go`

**Uses:**
- `internal/websterengine#reconcileHead`
- `internal/websterengine#headRefusal`

**Intent:**
Add the check that decides whether Go accepts the integration-fix strand's commits (#332b, decision D5's commit bound).

`integrationfix.go` opens with a file doc comment naming it as the home of the integration-fix attempt, and holds `checkFixCommits`.
It returns an error, making the attempt count as failed, unless all of these hold:
- `reportHead` equals `preFixHead` or descends from it;
- every commit in `preFixHead..reportHead` has exactly one parent;
- no commit in that range changes a path under `planDir` (taken relative to `worktree` when it lies inside it), the `lyxdirs.LyxDirName` entry itself, or a path under it;
- HEAD reconciles against `reportHead` under the existing merge-only rule, through `reconcileHead` with a new `integrationFixHeadRefusal` wording;
- the worktree carries no uncommitted or untracked change (`dirty`), since the post-fix verify runs over the working tree and must judge only the commits.

Each error names the offending commit, path or dirty state.
The warning is `reconcileHead`'s moved-HEAD notice, passed through.
List the range and each commit's changed paths with the git plumbing `gitwrap.go` already uses; add any helper it needs there (for example, a commit's parent count and changed paths), not in a new git layer.
`integrationFixHeadRefusal` lives beside `reportHeadRefusal` in `gitwrap.go`, worded for an attempt that fails rather than a verb to re-run: its way forward is the escalation's own.

Bound: the path rule binds commits only.
When `planDir` or `_lyx` lies outside the task repository, as in the hub geometry where `_lyx` is a directory link to the weft worktree, no commit can reach a file there, and only a commit adding or retargeting the `_lyx` entry itself is visible.
An on-disk plan write is caught instead by card 9's plan-fingerprint compare around the strand;
any other on-disk write under `_lyx` outside the repository is bounded by the prompt alone.

Tests in `integrationfix_test.go`, over temporary git repositories:
- non-merge commits touching ordinary files are accepted with an empty warning;
- no commits (`reportHead == preFixHead`) are accepted;
- a merge commit inside the range is refused;
- a commit touching a file under `_lyx`, one adding a `_lyx` entry at the root, and one touching the plan directory, are each refused and name the path;
- an uncommitted change or an untracked file left in the worktree is refused;
- a `reportHead` that does not descend from `preFixHead` is refused;
- a non-merge commit between `reportHead` and HEAD is refused;
- a clean parent merge between `reportHead` and HEAD is accepted with a warning.

**ImpactSummary:** adds one helper wording and any git helpers the check needs to `gitwrap.go`; existing callers of `reconcileHead` are untouched.
