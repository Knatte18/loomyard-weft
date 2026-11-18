# Decision record — fabric single-repo mode

## Goal

Let a hub run with one git repository: `_lym` and `.lym` are real directories in each worktree of the code repository,
the board lives on an orphan branch of that same repository, and there is no weft repository and no junction.
Everything outside fabric behaves as it does in a two-repository hub.
Only fabric, and `lymcwd` for anchor resolution, know which layout a hub has;
the shared helpers other modules gain (D9, D10) behave the same in both layouts.

## Scope

In:

- A hub layout "one repository" beside today's "two repositories", chosen at clone time and read from the worktree's tracked anchor marker.
- Clone, worktree add, commit, push, pull, replay, merge, teardown, archive and hand-over in the one-repository layout.
- Keeping records out of every parent branch: local merges, squash merges, merge-ins and pull requests.
- Code identity (tree, diff, dirtiness, changed files, held work) that ignores records, so gates, verify and landing judge code alone.
- A `hubforge` builder for a one-repository hub and tests of both layouts.
- Docs for setting up a one-repository hub.

Out:

- Converting an existing two-repository hub, or the reverse.
- Any change to two-repository hubs' behaviour.
- Rewriting `internal/fabricengine`; the mode is built into the current package (see Open risks).

## Decisions

### D1 — The layout is read from the tracked anchor marker

A worktree belongs to a one-repository hub when the anchor marker `.lym-anchor` sits at its own root.
In a two-repository hub the marker sits only at the weft root, never in a code worktree.
`lymcwd` reads the marker file from the worktree's root, and fabric reads it from the branch in git;
the predicate lives in `lymcwd`, which fabric already imports, so `lymcwd`'s import set is unchanged.
No config key, flag or hub-level file records the layout.

Rationale: the marker is tracked on every branch of a one-repository hub, so the layout reads right during clone once the prime is checked out,
and with `_bolt` missing or a weft sibling missing.

### D2 — Clone

`lym fabric clone` (`internal/fabriccli/clone.go`, `fabricengine.CloneHub`) takes the one-repository layout when:

- it is given one URL and that origin holds the Bolt branch `lym-bolt`: it clones the repository as the prime and adds `_bolt` as a worktree on `lym-bolt`;
- or it is given one URL and `--single-repo`, and origin holds no `lym-bolt`:
  it commits the anchor marker and seed records on the prime's branch first,
  then creates `lym-bolt` as an orphan branch seeded as a fresh weft bind is today, the shortname included.

`--single-repo` pushes the prime's marker commit before it creates `lym-bolt` on origin,
and pushes nothing further when that push is refused.
A refused clone removes the hub directory it made, so running the clone again starts clean.
The refusal names the push error; way forward: settle what it names, then run the clone again.
The protected-branch match moves from `protectedBranchRefusal` in `internal/landingshed/finalize.go`
into fabric's push classification (`internal/fabricengine/pushclassified.go`), and `landingshed` reads it from there.
A refusal that match recognises names the protection;
way forward: allow direct pushes to that branch for the account lym pushes as, then run the clone again.
The one-URL clone of an origin holding `lym-bolt` refuses when the prime's branch lacks `.lym-anchor` at its root, naming the missing marker;
way forward: commit at the root of origin's prime branch a `.lym-anchor` holding the anchor `lym-bolt`'s own `.lym-anchor` names, then run the clone again.

Bound: `--single-repo` creates only;
it refuses when origin already holds `lym-bolt`, holds a weft binding (`.lym-warp`), or a second URL is given.
It cannot recognise the code repository of an existing two-repository hub, whose origin holds neither;
run there, it starts a separate one-repository hub over the same code and changes nothing in the other hub.
The docs say so.
A single URL whose origin holds neither `lym-bolt` nor `--single-repo` keeps today's meaning: it is a weft URL.
`docs/getting-started.md` and `README.md` describe cloning a one-repository hub and what it puts on code branches,
and warn that switching, resetting or stashing in the prime holds or drops the prime's records (D7).

### D3 — Worktree layout and the paths that no longer run

In a one-repository hub:

- `_lym` and `.lym` are real directories at the worktree's anchor; `.lym-anchor` sits at the root, tracked on the branch.
- `Add` creates the task worktree on its branch and writes the origin record and anchor marker as a records commit on it;
  no sibling worktree or second branch exists.
  That records commit also deletes the parent's run-records root (`shedrun.RunsRootRel` under the anchor),
  and the root is never written to the new worktree's disk,
  as `createWeftWorktreeDroppingRuns` in `internal/fabricengine/forkrecords.go` does today.
- Junction wiring, junction repair (the junction-missing cause in `internal/fabricengine/reconcile.go`), `.lym` content adoption and unwire do not run.
- The shared `.git/info/exclude` gets `.lym`, the `.scratch` link and heddle's rendered files, through fabric's exclude mutator as today;
  it never gets `_lym`, `.lym-anchor`, an `_lym` line per anchor, or the weft-side artifact lines `internal/fabricengine/recordsgit.go` seeds, which would hide code paths.
- `.scratch` links to the worktree's own `.lym/scratch` through `internal/fslink`.
- The hub workspace file lists no sibling folders.
- `lymcwd.Resolve` reads the anchor from the worktree's own marker; `ResolveWithin` accepts a cwd at or below the anchor only.

### D4 — Records are committed on the code branch, beside agents' work

Fabric's records commit (`lym fabric commit`, `sync`, the transition commit, `webstercli`'s sync) commits on the worktree's own branch:

- It is path-limited, as `git commit -- <paths>` is, through `fabricengine.ScopedPathspec`, so code an agent has staged is never swept in.
- Fabric's empty records commits (`commitEmptySnapshot` in `internal/fabricengine/recordsgit.go`) land with the agent's staged entries left staged and none of them committed,
  as the path-limited commit does.
- Fabric's records and code commits in one worktree take one write lock.
- A commit that meets a held `index.lock` retries for a bounded time, a named constant, then fails as a commit failure does today.
- Its `Warp-SHA` trailer names HEAD before the commit; `Run-Transition` stays.
- `fabricengine.RecordsWorktree` and `fabricengine.RecordsRepoRoot` return the worktree and its repository,
  so weft-side readers outside fabric (`weavercli`, `battencli`'s catch-up, `reportcli`, `reportengine`) read the same repository unchanged.
- Archive and owner archive tags live in that repository.

Bound: an agent's own `git add -A` can put records paths into a code commit;
D5, D9 and D10 strip records by path, never by commit, so that is tolerated.

### D5 — Every merge keeps the receiving side's records

The records paths of a commit, or of a merge side, are the union of the anchors its own marker and its parent's marker name,
each as `<anchor>/_lym`, plus `.lym-anchor`.
D7's records-only test and D9's code identity read them the same way.

In a one-repository hub, every fabric verb that brings one branch's commits into another
(`fabricengine.Fabric.Merge` with or without squash, a merge-in from the parent into a task, a beam into its parent):

1. commits the receiving worktree's uncommitted records (D4);
2. merges without committing;
3. sets the records paths in both index and worktree to the receiving side's HEAD:
   restores what the source changed and deletes what only the source holds;
   before it does, it compares each path it restores against what step 2 wrote,
   and refuses the merge, as today's dirty refusal does, when a path changed in between;
4. hands the remaining code conflicts to today's handling, then commits with fabric's merge trailer, a named constant,
   and records the merge commit's SHA in the receiving worktree's `.lym`.

A merge whose source changed `.lym-anchor` other than through fabric's anchor-change commit refuses, naming the marker change;
way forward: revert the marker edit on the source branch, or make the anchor change through fabric, then merge again.

Fabric's own dirtiness guards (the merge verbs', `fabricengine.Fabric.FastForwardPair`'s) commit the worktree's records first in the same way and then judge code paths only.
A fast-forward of one branch onto another branch's commits is never made, since it would carry that branch's records; the verb makes a merge commit.
Bringing a branch level with its own origin tip first commits the worktree's records (D4),
then replays the local records-only commits onto origin's tip as D7 replays them;
it refuses only when a local non-records commit is unpushed.
An anchor change is a records commit that moves `_lym` from the old anchor to the new one and rewrites the marker in that one commit;
the anchor-change stop holds as today.

Bound: a records write that lands between step 1 and step 2 on a file the source changed makes git refuse the merge,
and one that lands between step 2 and step 3 on a path the source changed makes step 3 refuse it;
both read as today's dirty refusal.

Rationale: keeps `PATTERN-lym-files`' rule that records are per-branch and never a merge participant.

### D6 — Push, hand-over and teardown

- A pair push pushes the one branch once, under one push lock, so "records first, code only once it landed" holds trivially.
  The slug's teardown lock, the retry, the owed-push hold and the failed-push record with its creel notice stay;
  the failed-push record names the branch once, as its records side, so `PushOwed` and the act rule read it as today.
- In a one-repository hub no code tip can be owed apart from its records, because one push carries both.
  So the owed-code-tip push, the given-up code branch delete (`deleteGivenUpCodeBranch` in `internal/fabricengine/handover.go`)
  and `fabricengine.ReturnToPreviousTransition` do not run there.
  A rejected push or a takeover goes through D7's replay and today's hand-over only.
- A rejected push reads the run record at origin's fetched tip and hands over on another machine's takeover, as today.
- `fabricengine.OriginCodeTip` reads the trailer's commit, an ancestor of the pushed tip.
- Teardown tags the branch tip `archive/<slug>/<tip>`, pushes the tag, then deletes the branch locally and on origin, in today's order and locks;
  it deletes the D8 view ref too, and a view ref already gone on origin is no failure.

### D7 — Replay rewrites only records commits; the prime's records push is guarded

The predicates below are git reads:

- A records-only commit is a non-merge commit whose changed paths all lie under its records paths (D5), whatever trailer it carries.
- A merge commit fabric wrote is a merge commit carrying fabric's merge trailer whose SHA fabric recorded in the worktree's `.lym` when it wrote it (D5).
- A commit origin lacks is a commit not reachable from origin's ref of the branch being pushed or replayed.

A commit message alone never admits a commit that changes code paths.

In a one-repository hub, every replay of local commits onto origin's tip (pull, origin reconcile, rejected-push recovery, `internal/fabricengine/replay.go`),
on any code branch:

- replays only records-only commits, and sets aside ended-run commits only among those;
- rewrites a replayed commit's `Warp-SHA` trailer only when it names a commit being replayed,
  and then to the fork point, the newest commit both the local branch and origin hold,
  whose code is the code the replayed commit was made over; a `Warp-SHA` naming any other commit stays,
  so no trailer names code origin added and the commit never judged;
- when any other local commit is unpushed and origin diverged, rewrites nothing:
  a rejected push still reads a takeover as today, and otherwise the push or pull stops as owed, naming the commits.

`lym-bolt` keeps today's by-entry Bolt replay.

A records push of the prime's branch, which is also the operator's code branch, pushes only when every commit origin lacks is a records-only commit,
a merge commit fabric wrote, or a commit such a merge brings in through its second parent.
Otherwise it is owed: its failed-push record has a new class `operator_unpushed`, a named constant,
which `fabricengine.FailedPushesOwedBy` reads as owed and whose creel notice is sent as a refused push's is,
and its runs list the prime runs whose transition the push carries.
Its notice names the operator's unpushed commits and `git push` as the way forward,
and a run in the prime waits at its transition until the operator pushes.

A prime records push that origin refuses as a protected branch, as fabric's push classification recognises it (D2),
records a failed push of a new class `protected`, a named constant the act rule does not read as owed, so no run waits on it.
Once such a record exists for the prime's branch, the guard's refusal is recorded as `protected` too.
The notice names the protection; way forward: allow direct pushes to the branch for the account lym pushes as, after which the next records push of the prime carries the records.

The prime's live records sit on the operator's checked-out branch.
`lym fabric clone` records the prime's branch in the prime's `.lym`, which no checkout changes.
Fabric refuses to commit records in the prime while it is not on that branch, naming both branches, and leaves the records uncommitted on disk;
way forward: switch the prime back to the recorded branch, with `git switch --merge` when git refuses to carry the uncommitted records.

Bound: fabric rewrites only its own unpushed records-only commits and never a code commit.
The guard binds records pushes; a landing's push of its parent carries the operator's unpushed commits on that branch, as it does today in both layouts.
While the prime's branch refuses direct pushes, the prime's records stay unpushed and no other machine sees them.
While the prime is off its recorded branch, every prime run's records commit refuses, so prime runs stall until the switch back.
A `git reset` or `git stash` in the prime can drop its uncommitted live records.
The docs D2 requires warn of both.

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
- Fabric's push of a branch that has a local view ref refreshes the view against the origin record's `parent_branch`; a failed view push leaves a failed-push record that holds no run, and the publish before a merge retries it.
- After GitHub merges a view, Finalize makes no local merge of the task:
  it brings the local parent level with origin's parent tip by D5's rule, and the landed judgment is D9's held-work check.

Bound: the verb force-pushes only `lym-view/` refs, never a code or Bolt branch;
review comments on a refreshed view read as outdated; GitHub calls stay with `internal/githubclient`'s callers.

### D9 — Code identity ignores the records paths, inside fabric and out

Every judgment of code by tree, diff, dirtiness, changed files or held work excludes the records paths (D5),
through one helper built on `lymdirs`, in a leaf package its readers may import.
It applies in both layouts; in a two-repository hub the paths are absent from the code tree, so nothing changes there.

- Tree identity: `verifytree.Verify`'s HEAD tree and the gate green record's (`internal/gatecli/green.go`) use the stripped tree.
- Dirtiness and changed files: `verifytree.DirtyPaths`, gate scope, weaver verify, drift, the landed size and the lint diffs leave records out.
- Held work: `fabricengine.Fabric.HoldsWork`, `fabricengine.ParentHoldsTaskWork`, `fabricengine.OriginParentHoldsTaskWork` and the landing and catch-up checks built on them
  read "the parent holds the task's work" as "merging the task's code into the parent changes no code".
- SHA readers: a check that HEAD is unchanged, meaning no code changed, becomes a code-tree check;
  a check that HEAD moved, meaning code was committed (webster's card checks in `internal/websterengine/gitwrap.go`), requires the code tree to change.
- Walks of the commits between two SHAs that allow only certain commits (webster's `reconcileHead` in `internal/websterengine/gitwrap.go`):
  records-only commits, as D7 defines them, pass such a walk as a clean parent merge does.
- The local-versus-upstream tip relation (`fabricengine.Fabric.ReadParentTips`, Finalize's parent alignment) judges code commits only,
  so unpushed records-only commits leave a parent equal to or behind its upstream.
- Ancestry checks that ask whether a commit is in a branch's history stay.

Plan enumerates the readers of each kind.

### D10 — Webster's write audit classifies records the same in both layouts

`internal/websterengine/findingtext.go` and `internal/websterengine/audit.go` test a write against the records paths before the worktree-root case,
so a write under `_lym` gets the same verdict whether `_lym` is a link or a real directory.

### D11 — The board on `lym-bolt`

The Bolt checkout `_bolt` is a worktree of the one repository on `lym-bolt`.
`boardengine` writes through the Bolt handle as today; the Bolt pushes and pulls `lym-bolt`.
The render-manifest `.gitignore` stays on `lym-bolt`, which no code branch merges.

### D12 — Tests and fixtures

`hubforge` builds a one-repository hub beside its two-repository hub.
The acceptance scenarios run in the one-repository layout; the two-repository tests stay as they are.

### Added after Discussion (parent, 2026-11-18): Build the mode into the current fabricengine (F16)

Decision: Design the single-repo mode as a build into the current internal/fabricengine, as the record does now. It is not a spec for a future rewrite.

Rationale: The operator judged a full fabricengine rewrite too large on 2026-11-18, and the clean-room rewrite note was removed from the board; no rewrite is planned.

## Constraints

- The operator ruled that `_lym` is never in the exclude (board entry, 2026-11-18), and lym never force-adds (`PATTERN-gitrepo-client-boundary`).
- `PATTERN-fabric-boundary`: every git write on the one repository goes through `internal/fabricengine`; D8's verb is a general seam that knows nothing of pull requests.
- `PATTERN-gitrepo-client-boundary`: D8's force-push and every fetch and push go through `gitrepo` under the remote-refs lock.
- `PATTERN-lym-files`: `lymdirs` stays the only declarer of `_lym` and `.lym`.
- `PATTERN-cwd-resolution`: only `lymcwd` resolves the anchor; only fabric writes the marker or reads it from git.
- `PATTERN-code-conventions`: `lym-bolt`, `lym-view/`, the merge trailer, the `index.lock` retry bound,
  the `operator_unpushed` and `protected` failed-push classes and the view commit's identity and dates are named constants.
- The Webster card that adds `PATTERN-single-repo-hub` also adds a one-repository paragraph to the backgrounds of `PATTERN-fabric-boundary` (excludes, Bolt `.gitignore`, teardown),
  `PATTERN-lym-files`, `PATTERN-cwd-resolution` (marker at the worktree root) and `PATTERN-pair-remote` (one push, guarded prime push);
  the overview lines of `PATTERN-fabric-boundary`, `PATTERN-lym-files` and `PATTERN-pair-remote` stay.
- Tool facts the Decisions rest on, each with its pin:
  - D4: `git commit -- <paths>` commits only those paths and leaves other staged entries staged.
    Pin: the `git-commit` documentation (`--only`, the default with paths),
    and `TestCommitPathAllowEmpty`'s row `UnchangedPathWithAnotherPathStaged` (`internal/gitrepo/commitpathallowempty_integration_test.go`).
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

- Board note `clean-room-module-rewrite`, and `design-simplify-callers`: the entry says the operator wants this mode taken in if fabricengine is rewritten, and both notes aim to shrink the package this record grows.
  This record builds the mode into the current package now. Whether the parent wants that, or a spec for the rewrite only, is the parent's call.
- D7 puts records commits on the operator's code branch in the prime; an operator who rebases that branch rewrites fabric's records commits too.

## Acceptance criteria

- D1: a worktree with `.lym-anchor` at its root reads as one repository, with `_bolt` missing too; a two-repository worktree with its weft sibling missing still reads as two.
- D2: clone of a URL whose origin holds `lym-bolt` makes a one-repository hub; `--single-repo` on a fresh origin commits the marker on the prime's branch, pushes it, then creates and pushes `lym-bolt`; `--single-repo` refuses an origin holding `lym-bolt` or a weft binding, and a second URL; a `--single-repo` whose marker push is refused, as a protected branch or otherwise, leaves no `lym-bolt` on origin and no hub directory, and the protected refusal names the protection; a one-URL clone of an origin holding `lym-bolt` whose prime branch lacks `.lym-anchor` refuses, naming the missing marker.
- D3: `Add` in a one-repository hub leaves `_lym` and `.lym` as real directories, `.lym-anchor` tracked at the root, `.lym` excluded, no `_lym` or weft-artifact exclude line, and no sibling worktree; a task forked from a parent carrying run records starts with none; `lym fabric reconcile` reports no junction cause; `lymcwd.Resolve` returns the anchor from the marker.
- D4: a records commit with code staged by another process commits only records paths, leaves the code staged, and names the prior HEAD in `Warp-SHA`; an empty records commit made while code is staged lands and leaves the code staged; a fabric records commit and a fabric code commit started together in one worktree run one after the other; a held `index.lock` released within the bound lets the commit land; `fabricengine.RecordsRepoRoot` returns the one repository.
- D5: a local merge and a squash merge of a task into its parent, with uncommitted parent records and conflicting records edits on both sides, leave the parent's records equal to its own pre-merge records in index and worktree, with no task-only records file; a merge-in from the parent keeps the task's records; a merge whose source is a fast-forward of the receiver makes a merge commit carrying the merge trailer; an anchor-change commit merged into the parent changes no parent code path; a merge whose source edited `.lym-anchor` to another anchor refuses, naming the marker change; a receiving-side records write between step 2 and step 3 on a path the source changed makes the merge refuse.
- D6: a pair push pushes the branch once; a refused push leaves one failed-push record read as owed; after a takeover in a one-repository hub, origin's branch keeps every transition commit; teardown pushes the archive tag before deleting the branch on origin, deletes the view ref, and succeeds when origin already lost the view ref.
- D7: a pull over a diverged task branch with local records-only commits replays them with each trailer that named a replayed commit rewritten to the fork point; with a code commit among origin's new commits it replays them all the same, and no trailer names that commit; with a local code commit it rewrites nothing and stops naming it; a commit changing code paths whose message carries fabric's merge trailer is neither replayed nor admitted by the prime guard; a prime records push with an operator commit unpushed, including one fast-forwarded from a branch origin holds, pushes nothing and leaves an `operator_unpushed` failed push naming that commit and listing the prime runs, its notice is sent, and a run in the prime with that push owed does not act past its transition until the push lands; a prime records push refused as a protected branch leaves a `protected` failed push that holds no run; a records commit in a prime switched off its recorded branch refuses, naming both branches, and leaves the records on disk.
- D8: the view's three-way merge into the parent changes the parent's code as the branch does and leaves its records unchanged; two publishes of an unchanged branch leave the view ref on the same SHA and push nothing the second time, and a publish after origin lost the view ref pushes it again; a publish against a base other than the origin record's `parent_branch` refuses, naming both; a push after a new commit refreshes the view; after a GitHub merge of the view, Finalize brings the local parent level, makes no merge commit, and reads the task as landed; an already merged beam pull request whose head is the published view reads as landed; with uncommitted and unpushed parent records, a pull-request landing and a local landing both complete; in a two-repository hub the verb returns the branch and pushes nothing.
- D9: a records-only commit leaves the verify tree identity and the gate green record valid; uncommitted records edits do not make `verifytree.DirtyPaths` report dirt; changed-file sets omit records paths; `fabricengine.Fabric.HoldsWork` reads true for a parent that merged the task's code with D5's strip, and after a squash landing whose parent and task records both changed since the fork; a webster card check does not accept a records-only commit as code work; a records commit made during a webster batch does not refuse the batch.
- D10: a webster write under `_lym` gets the same audit verdict in both layouts.
- D11: a board write in a one-repository hub commits and pushes on `lym-bolt` and changes no code branch.
- D12: `hubforge` builds a one-repository hub that the D1 to D11 scenarios run on.
- Review: D2: `docs/getting-started.md` and `README.md` describe cloning a one-repository hub, what it puts on code branches, the `--single-repo` hazard on a two-repository hub's code repository, and the hazards of switching, resetting or stashing in the prime.

## PATTERN changes

- `PATTERN-single-repo-hub` — adds: Running a hub as one repository: records on each code branch, the board on `lym-bolt`; merges, pull request views and code identity strip the records paths.
- `PATTERN-cwd-resolution` — changes: the overview line reads the anchor from the worktree's own `.lym-anchor` at its records root, the weft root in a two-repository hub and the worktree root in a one-repository hub.

## Notes for the plan writer

- `internal/fabricengine/merge.go`'s package comment says the weft is not a merge participant; D5 is how that stays true with one repository.
- The fabric-internal held-work check is in `internal/fabricengine/parenttips.go`.
  The held-work checks strip the records paths from the merge base and from both sides before the in-memory merge,
  so records conflicts cannot make the merge unclean; then they compare the code trees.
