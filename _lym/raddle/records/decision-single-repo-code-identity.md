## Goal

Records-free code identity and pull request views for one-repository hubs

Takes: single-repo-fabric-mode's exports: fabricengine.RecordsPathsAt, the records-only-commit and fabric-written-merge-commit predicates, the merge trailer constant, the lym-view/ ref prefix constant and hubforge's one-repository hub; landingshed after landingshed-design-simplify lands; verifytree.Verify and DirtyPaths, gatecli's green record and testmap head reads, gitrepo's ChangedFilesSince and WorktreeChangedFiles, weaverengine's verify, driftengine's head and dirty reads, websterengine's gitwrap.go card checks and reconcileHead, findingtext.go and audit.go, commentlint's diff, loomshed's and loomcli's dirty readers, landingshed's Finalize, finalize_pr.go, prreview.go and parentalign.go, beamcli's pr verb, pullrequest's merge-state head check, sheddriver's and shedarbiter's head comparisons, fabricengine's HoldsWork, ParentHoldsTaskWork, OriginParentHoldsTaskWork and ReadParentTips
Delivers: a leaf package on lymdirs that strips given records paths from trees, diffs and path sets and reads no git; every code identity judgment by tree, diff, dirtiness, changed files or held work excludes the records paths in both layouts; held work read as merging the task's code changes no code; webster's write audit tests records paths before the worktree-root case; fabric's general view-publish verb, its idempotent lym-view/<branch> commit and force-push, the view refresh on a pair push, and PR-Review, Finalize's pull-request path and lym beam pr opening, comparing and recording pull requests on the view tip
Exports: the leaf package's records-path strip functions; fabricengine's view-publish verb returning the pull request head ref

## Scope

The part changes these packages:

- internal/recordspaths (about 200 changed lines)
- internal/fabricengine (about 900 changed lines)
- internal/landingshed (about 600 changed lines)
- internal/websterengine (about 400 changed lines)
- internal/driftengine (about 250 changed lines)
- internal/driftcli (about 30 changed lines)
- internal/verifytree (about 200 changed lines)
- internal/gatecli (about 200 changed lines)
- internal/gitrepo (about 150 changed lines)
- internal/beamcli (about 150 changed lines)
- internal/loomcli (about 120 changed lines)
- internal/weaverengine (about 80 changed lines)
- internal/weavercli (about 40 changed lines)
- internal/commentlint (about 60 changed lines)
- internal/loomshed (about 60 changed lines)
- internal/pullrequest (about 60 changed lines)
- internal/sheddriver (about 40 changed lines)
- internal/shedarbiter (about 40 changed lines)

## Decisions

### D8 — A pull request's head is a records-free view

Fabric gains one general verb that publishes a view of a branch against a given base branch and returns the ref a pull request uses as its head.

- Two repositories: it returns the branch and pushes nothing.
- One repository: it writes one commit whose parent is the merge base of branch and base,
  whose tree is the branch's tree with the records paths taken from the merge base,
  and force-pushes it to `lym-view/<branch>` under the slug's teardown lock, shared as for a pair push.
  The caller passes the branch the pull request targets as the base;
  the verb refuses when that branch differs from the branch's `parent_branch` in its fabric origin record, naming both;
  way forward: retarget the pull request to `parent_branch` on GitHub, then run the step again.
- The publish is idempotent.
  It reuses origin's `lym-view/<branch>` tip, fetched first, when that tip's tree and parent equal the ones it would write, and pushes nothing;
  when origin lacks the view ref, it pushes.
  Otherwise it writes the commit with a fixed author and committer identity and fixed dates, named constants,
  so the same inputs give the same SHA.
- PR-Review, Finalize's pull-request path (`internal/landingshed/finalize_pr.go`) and `lym beam pr` publish before opening, updating or merging a pull request, and open it on the returned ref.
- Every place that compares, records or passes a pull request's head uses the view tip the publish returned:
  Finalize's merge's expected head, the recorded and gated head, the merged-head check of a recorded beam landing and `lym beam pr`'s pushed head.
  A merged pull request counts as holding the task's work when its head is the view published from the task tip,
  or when D9's held-work check reads true; commit ancestry is never used.
- Fabric's push of a branch that has a local view ref refreshes the view against the origin record's `parent_branch`.
  A failed view push leaves a failed-push record of a class `fabricengine.FailedPushesOwedBy` does not read as owed,
  and is never returned in the pair push's error, so a transition's push result is the branch push alone;
  the publish before a merge retries it.
- After GitHub merges a view, Finalize makes no local merge of the task:
  it brings the local parent level with origin's parent tip by D5's rule, and the landed judgment is D9's held-work check.

Bound: the verb force-pushes only `lym-view/` refs, never a code or Bolt branch;
review comments on a refreshed view read as outdated; GitHub calls stay with `internal/githubclient`'s callers.

### D9 — Code identity ignores the records paths, inside fabric and out

Every judgment of code by tree, diff, dirtiness, changed files or held work excludes the records paths (D5),
through one helper built on `lymdirs`, in a leaf package its readers may import.
The helper takes the records paths, or the anchors they derive from, as input and reads no git.
A reader outside fabric gets a commit's records paths from one fabric read, `fabricengine.RecordsPathsAt` (new),
a general seam that reads the commit's marker and its parent's, or is handed them by its caller.
It applies in both layouts; in a two-repository hub the paths are absent from the code tree, so nothing changes there.

- Tree identity: `verifytree.Verify`'s HEAD tree and the gate green record's (`internal/gatecli/green.go`) use the stripped tree.
- Dirtiness and changed files: `verifytree.DirtyPaths`, gate scope, weaver verify, drift, the landed size and the lint diffs leave records out.
- Held work: `fabricengine.Fabric.HoldsWork`, `fabricengine.ParentHoldsTaskWork`, `fabricengine.OriginParentHoldsTaskWork` and the landing and catch-up checks built on them
  read "the parent holds the task's work" as "merging the task's code into the parent changes no code".
- SHA readers: a check that HEAD is unchanged, meaning no code changed, becomes a code-tree check;
  a check that HEAD moved, meaning code was committed (webster's card checks in `internal/websterengine/gitwrap.go`), requires the code tree to change.
- Walks of the commits between two SHAs that allow only certain commits (webster's `reconcileHead` in `internal/websterengine/gitwrap.go`):
  records-only commits, as D7 defines them, pass such a walk as a clean parent merge does.
  Merge commits fabric wrote, as D7 defines them, pass it too:
  D7's test over the parents' records-stripped trees takes the place of the walk's clean-merge tree test,
  and the walk's other checks on a merge stay.
- The local-versus-upstream tip relation (`fabricengine.Fabric.ReadParentTips`, Finalize's parent alignment) judges code commits only,
  so unpushed records-only commits leave a parent equal to or behind its upstream.
- Ancestry checks that ask whether a commit is in a branch's history stay.

Plan enumerates the readers of each kind.

### D10 — Webster's write audit classifies records the same in both layouts

`internal/websterengine/findingtext.go` and `internal/websterengine/audit.go` test a write against the records paths before the worktree-root case,
so a write under `_lym` gets the same verdict whether `_lym` is a link or a real directory.

### Added after Discussion (parent, 2026-11-18): Build the mode into the current fabricengine (F16)

Decision: Design the single-repo mode as a build into the current internal/fabricengine, as the record does now. It is not a spec for a future rewrite.

Rationale: The operator judged a full fabricengine rewrite too large on 2026-11-18, and the clean-room rewrite note was removed from the board; no rewrite is planned.

### Added after Discussion (operator, 2026-11-18): The motif writes the design only

Decision: This motif changes no code. Its record is written as the design a later task builds into the current internal/fabricengine.

Rationale: Operator, 2026-11-18: the motif rewrites nothing, but writes as if the work will be done.

## Constraints

- The operator ruled that `_lym` is never in the exclude (board entry, 2026-11-18), and lym never force-adds (`PATTERN-gitrepo-client-boundary`).
- `PATTERN-fabric-boundary`: every git write on the one repository goes through `internal/fabricengine`; D8's verb is a general seam that knows nothing of pull requests.
- `PATTERN-gitrepo-client-boundary`: D8's force-push and every fetch and push go through `gitrepo` under the remote-refs lock.
- `PATTERN-lym-files`: `lymdirs` stays the only declarer of `_lym` and `.lym`.
- `PATTERN-cwd-resolution`: only `lymcwd` resolves the anchor; only fabric writes the marker or reads it from git.
- `PATTERN-code-conventions`: `lym-bolt`, `lym-view/`, the merge trailer, the `index.lock` retry bound,
  the `operator_unpushed` and `protected` failed-push classes, the `lym.primeBranch` (new) config key
  and the view commit's identity and dates are named constants.
- The Webster card that adds `PATTERN-single-repo-hub` also adds a one-repository paragraph to the backgrounds of `PATTERN-fabric-boundary` (excludes, Bolt `.gitignore`, teardown),
  `PATTERN-lym-files`, `PATTERN-cwd-resolution` (marker at the worktree root), `PATTERN-pair-remote` (one push, guarded prime push)
  and `PATTERN-run-fsm` (the owed `operator_unpushed` class and the `protected` exception, in "An owed push holds the run");
  the overview lines of `PATTERN-fabric-boundary`, `PATTERN-lym-files` and `PATTERN-pair-remote` stay.
- Tool facts the Decisions rest on, each with its pin:
  - D4: `git commit -- <paths>` commits only those paths and leaves other staged entries staged.
    Pin: the `git-commit` documentation (`--only`, the default with paths),
    and `TestCommitPathAllowEmpty`'s row `UnchangedPathWithAnotherPathStaged` (`internal/gitrepo/commitpathallowempty_integration_test.go`).
  - D4: `git commit -- <paths>` during a merge fails with `fatal: cannot do a partial commit during a merge.`,
    and during a cherry-pick or a rebase with the matching message.
    Pin: the merge case observed with git in a scratch repository in review round 2
    and stated in `failureDisposition`'s doc in `internal/statuscommit/statuscommit.go`;
    git's `builtin/commit.c` holds all three messages.
  - D4: a merge commit takes HEAD as it stands at commit time as its first parent,
    and `git merge --abort` resets to HEAD, so a commit moved onto HEAD during a merge survives both.
    Pin: the `git-merge` documentation (`--abort` is `git reset --merge` while `MERGE_HEAD` is present);
    the D4 acceptance scenario is its observed result.
  - D5: `git merge --no-commit --no-ff`, then restoring paths from HEAD in index and worktree and removing source-only paths, resolves the conflicts on those paths.
    Pin: the `git-merge` documentation (`--no-commit`) and the `git-restore` documentation (`--staged --worktree --source`);
    the D5 acceptance scenario is its observed result.
  - D8: a GitHub pull request shows and merges the diff of its head against the merge base with its base branch,
    so a head parented on that merge base shows and merges only the view's changes.
    Pin: GitHub's documentation on comparing branches in pull requests (three-dot diff); the D8 acceptance scenario is its observed result.
  - D6: GitHub deletes a merged pull request's head branch only under the repository's automatic head-branch deletion setting.
    Pin: GitHub's documentation on managing the automatic deletion of branches.

## Auto-mode assumptions

- D4: records on the code branch, auto-picked.
- D1: the layout read from the tracked marker, auto-picked.
- D5: strip the records paths at every merge, auto-picked.
- D7: records-only replay and the guarded prime push, auto-picked.
- D8: pull requests on a records-free view, auto-picked.
- D9: code identity that ignores records in both layouts, auto-picked.
- D11: the Bolt on orphan `lym-bolt`, auto-picked.
- Scope Out: conversion excluded, auto-picked.

## Open risks

## Acceptance criteria

- D8: the view's three-way merge into the parent changes the parent's code as the branch does and leaves its records unchanged; two publishes of an unchanged branch leave the view ref on the same SHA and push nothing the second time, and a publish after origin lost the view ref pushes it again; a publish against a base other than the origin record's `parent_branch` refuses, naming both; a push after a new commit refreshes the view; a transition whose view refresh fails leaves a failed-push record, and the run acts past that transition once its branch push lands; after a GitHub merge of the view, Finalize brings the local parent level, makes no merge commit, and reads the task as landed; an already merged beam pull request whose head is the published view reads as landed; with uncommitted and unpushed parent records, a pull-request landing and a local landing both complete; in a two-repository hub the verb returns the branch and pushes nothing.
- D9: a records-only commit leaves the verify tree identity and the gate green record valid; uncommitted records edits do not make `verifytree.DirtyPaths` report dirt; changed-file sets omit records paths; `fabricengine.Fabric.HoldsWork` reads true for a parent that merged the task's code with D5's strip, and after a squash landing whose parent and task records both changed since the fork; a webster card check does not accept a records-only commit as code work; a records commit made during a webster batch does not refuse the batch; a parent merge-in made during a webster batch, with conflicting records on both sides, does not refuse the batch.
- D10: a webster write under `_lym` gets the same audit verdict in both layouts.
