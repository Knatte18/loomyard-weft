## Goal

One-repository layout in fabric: clone, worktrees, records commits, merges, pushes and replay

Takes: the decision record's D1 to D7, D11 and D12; fabricengine's CloneHub (clone.go), Add and createWeftWorktreeDroppingRuns (add.go, forkrecords.go), junction wiring and adoption (junction.go, weftwiring.go), the exclude mutator (gitexclude.go, recordsgit.go's seedWeftArtifactExcludes), scratchlink.go, hubworkspace.go, the records commit (commit.go, recordsgit.go, trailercommit.go), the merge (merge.go, mergelifecycle.go, mergestate.go, mergestateactive.go, parenttips.go's FastForwardPair), the push (pushanchored.go, pushclassified.go, failedpush.go, handover.go, codetipsettle.go, originreads.go), replay.go, pull.go, originreconcile.go, archive and teardown, bolt.go and anchorchange.go; lymcwd's readRecordedAnchor, Resolve and ResolveWithin; statuscommit's mid-merge skip and failureDisposition; landingshed's protectedBranchRefusal and Finalize's mid-merge way forward; hubforge's two-repository builder
Delivers: the one-repository layout read from .lym-anchor at the worktree root; lym fabric clone by one URL over an origin holding lym-bolt and with --single-repo; Add, the path-limited records commit under the worktree write lock with its index.lock retry and mid-merge route, D5's records strip at every merge, the one-branch pair push, teardown, the records-only replay and the guarded prime push with the lym.primeBranch key in that layout; statuscommit's wait in place of its mid-merge skip and its completion on protected-only push failures; the Bolt on orphan lym-bolt; hubforge's one-repository hub with the D1 to D7, D11 and D12 scenarios; the docs D2 names; PATTERN-single-repo-hub, the PATTERN-run-fsm and PATTERN-cwd-resolution changes and the background paragraphs the record's Constraints list
Exports: lymcwd's one-repository predicate; fabricengine.RecordsPathsAt, a commit's records paths as D5 defines them; fabric's records-only-commit and fabric-written-merge-commit predicates as D7 defines them, and the merge trailer constant; the lym-view/ ref prefix constant, whose ref teardown deletes and the next part publishes; the operator_unpushed and protected failed-push class constants; the lym.primeBranch key constant; hubforge's one-repository hub builder

## Scope

The part changes these packages:

- internal/fabricengine (about 4500 changed lines)
- internal/hubforge (about 600 changed lines)
- internal/lymcwd (about 250 changed lines)
- internal/statuscommit (about 200 changed lines)
- internal/fabriccli (about 150 changed lines)
- internal/landingshed (about 60 changed lines)
- docs (about 90 changed lines)
- README.md (about 60 changed lines)
- pattern (about 80 changed lines)

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
- A records commit never skips while the worktree is mid-merge.
  `fabricengine.MergeStateActive` (`internal/fabricengine/mergestateactive.go`) reads the one worktree there,
  and git refuses a path-limited commit during a merge, a cherry-pick or a rebase (Constraints).
  While `MERGE_HEAD` is present, fabric writes the records commit from a temporary index seeded from HEAD
  with the records paths taken from the worktree, moves HEAD onto it,
  and stages those records paths in the worktree's index;
  concluding the merge then carries them.
  It does so only while the index holds every records path as HEAD does.
  It holds the write lock above, which fabric's merge abort takes too,
  and moves the start fabric's merge record holds (`WarpStart`) onto the new commit,
  so `fabricengine.Fabric.MergeAbort` resets to that commit,
  and its conclude-landed guard does not read it as a landed conclude;
  `git merge --abort` keeps it as well.
  In a one-repository hub, `fabricengine.Fabric.MergeAbort` first commits the worktree's records the same way,
  or by the rule below for a merge stopped before D5 step 3.
  A merge fabric started that stopped before D5 step 3, as fabric's merge record notes, holds no conflict resolution yet,
  and its index holds the records step 2 took from the source, so the route above refuses it.
  A records commit that finds one, and `fabricengine.Fabric.MergeAbort` on one, build the commit from a temporary index seeded from HEAD
  with the worktree's content of every records path whose index entry equals HEAD's or that neither holds, a new file,
  and HEAD's version of every other records path;
  they move HEAD and `WarpStart` onto it as above, and only then reset to it as the abort does,
  all under one hold of the write lock, so no records write lands between the commit and the reset.
  The reset is hard, so the records the commit carries stay in HEAD and the worktree.
  A records path whose index entry differs from HEAD's, or is unmerged, is one step 2 wrote from the source:
  its worktree holds step 2's result, so the commit keeps HEAD's version.
  `fabricengine.MergeStateActive` reports any other merge whose index holds a records path unlike HEAD's,
  and a cherry-pick, revert or rebase in progress, since aborting one of those drops a commit made during it.
  The transition seam (`internal/statuscommit`) then waits in place of its skip, re-probing as it retries an owed push,
  with a warning naming the operation and its way forward: conclude or abort it.
  A run never acts past a transition whose records commit has not landed.
- Archive and owner archive tags live in that repository.

Bound: an agent's own `git add -A` can put records paths into a code commit;
D5, D9 and D10 strip records by path, never by commit, so that is tolerated.
A records write made after a fabric merge stopped before D5 step 3, to a records path the source changed,
is lost when that merge is aborted.

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
4. hands the remaining code conflicts to today's handling, then commits with fabric's merge trailer, a named constant.

Steps 1 to 3 hold D4's write lock, so no records commit lands while the index holds the source's records;
step 4 releases it before it hands conflicts over and takes it again for its commit.
Fabric's merge record notes when step 3 is done.
A crash before step 3 leaves a merge that the next records commit, or `lym fabric merge --abort`, commits around and aborts (D4);
the abort's reset drops the source's records step 2 wrote and keeps that records commit.
A crash after it leaves today's parked merge, which records commits pass through by D4.
A merge verb run again over either refuses with today's already-in-progress guard and never aborts the merge;
it checks that guard before step 1 and before its dirtiness guards commit records, so neither aborts the merge first.
Way forward: `lym fabric merge --abort`, or for a merge past step 3, `lym fabric merge --continue` once its conflicts are resolved.
`lym fabric merge --continue` on a merge stopped before step 3 refuses, naming the unfinished records step;
way forward: `lym fabric merge --abort`, then merge again.
Fabric's mid-merge read tells a merge stopped before step 3 apart from a parked one,
and Finalize's way forward out of a fabric merge in the parent pair names only `lym fabric merge --abort` for it.
`mergeresolve`'s abort of an in-progress merge before its attempt, and the landing's pre-merge abort of the row's own parked merge-in,
stay as they are and are the only paths that abort a merge before merging again.

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
- A merge commit fabric wrote is a merge commit carrying fabric's merge trailer whose tree equals its first parent's records paths
  over the clean merge of its two parents' records-stripped trees (D5).
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
a merge commit fabric wrote, or a commit such a merge brings in through a second parent that some origin ref holds.
Otherwise it is owed: its failed-push record has a new class `operator_unpushed`, a named constant,
which `fabricengine.FailedPushesOwedBy` and `fabricengine.IsPushOwed` read as owed and whose creel notice is sent as a refused push's is,
and its runs list the prime runs whose transition the push carries.
Its notice names the operator's unpushed commits and `git push` as the way forward,
and a run in the prime waits at its transition until the operator pushes.

A prime records push that origin refuses as a protected branch, as fabric's push classification recognises it (D2),
records a failed push of a new class `protected`, a named constant
that neither `fabricengine.FailedPushesOwedBy` nor `fabricengine.IsPushOwed` reads as owed.
Once such a record exists for the prime's branch, the guard's refusal is recorded as `protected` too.
The transition seam (`internal/statuscommit`) completes a transition whose push fails with `protected` failures only,
logging a warning that names the protection, so the run acts with that transition's records unpushed (PATTERN changes).
The notice names the protection; way forward: allow direct pushes to the branch for the account lym pushes as, after which the next records push of the prime carries the records.

The prime's live records sit on the operator's checked-out branch.
`lym fabric clone` records the prime's branch in the prime repository's git config key `lym.primeBranch` (new), a named constant,
which no checkout changes and which a lost `.lym` keeps.
Fabric refuses to commit records in the prime while it is not on that branch, naming both branches, and leaves the records uncommitted on disk;
way forward: switch the prime back to the recorded branch, with `git switch --merge` when git refuses to carry the uncommitted records.
When the key is absent, fabric refuses to commit records in the prime, naming the key, and leaves the records on disk;
way forward: run `git config lym.primeBranch <branch>` in the prime, naming the branch its records live on, then run the step again.

Bound: fabric rewrites only its own unpushed records-only commits and never a code commit.
The guard binds records pushes; a landing's push of its parent carries the operator's unpushed commits on that branch, as it does today in both layouts.
A squash landing's commit, and a merge fabric wrote over hand-resolved code conflicts,
do not read as a merge commit fabric wrote;
the guard owes each as `operator_unpushed` until it reaches origin, through a landing's push of its parent or the operator's `git push`.
While the prime's branch refuses direct pushes, the prime's records stay unpushed, no other machine sees them,
and prime runs act past transitions whose records are unpushed.
While the prime is off its recorded branch, every prime run's records commit refuses, so prime runs stall until the switch back.
A `git reset` or `git stash` in the prime can drop its uncommitted live records.
The docs D2 requires warn of both.

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

- D7 puts records commits on the operator's code branch in the prime; an operator who rebases that branch rewrites fabric's records commits too.

<!-- lym:carry-over Code-Review -->
- Code-Review, round 1: closed by an accepted circling decision; a finding below was addressed by the round's fixer or is still unfixed.
  Review: _lym/reviews/code/round-1-review.md; fixer report: _lym/reviews/code/round-1-fixer-report.md.
  - add-anchor-does-not-move-forked-records (class design, severity MEDIUM, open in rounds 1)
  - battencli-one-repo-seam-untested (class design, severity MEDIUM, open in rounds 1)
  - merge-over-parked-merge-pinned-for-mergein-only (class design, severity MEDIUM, open in rounds 1)
  - origin-reconcile-one-repo-code-stop-untested (class design, severity MEDIUM, open in rounds 1)
  - readme-d2-clone-description-missing (class design, severity BLOCKING, open in rounds 1)
  - replay-nonreplayed-warp-sha-untested (class design, severity MEDIUM, open in rounds 1)
  - run-fsm-background-protected-contradicts-d7 (class consistency, severity MEDIUM, open in rounds 1)
<!-- /lym:carry-over Code-Review -->

## Acceptance criteria

- D1: a worktree with `.lym-anchor` at its root reads as one repository, with `_bolt` missing too; a two-repository worktree with its weft sibling missing still reads as two.
- D2: clone of a URL whose origin holds `lym-bolt` makes a one-repository hub; `--single-repo` on a fresh origin commits the marker on the prime's branch, pushes it, then creates and pushes `lym-bolt`; `--single-repo` refuses an origin holding `lym-bolt` or a weft binding, and a second URL; a `--single-repo` whose marker push is refused, as a protected branch or otherwise, leaves no `lym-bolt` on origin and no hub directory, and the protected refusal names the protection; a one-URL clone of an origin holding `lym-bolt` whose prime branch lacks `.lym-anchor` refuses, naming the missing marker.
- D3: `Add` in a one-repository hub leaves `_lym` and `.lym` as real directories, `.lym-anchor` tracked at the root, `.lym` excluded, no `_lym` or weft-artifact exclude line, and no sibling worktree; a task forked from a parent carrying run records starts with none; `lym fabric reconcile` reports no junction cause; `lymcwd.Resolve` returns the anchor from the marker.
- D4: a records commit with code staged by another process commits only records paths, leaves the code staged, and names the prior HEAD in `Warp-SHA`; an empty records commit made while code is staged lands and leaves the code staged; a fabric records commit and a fabric code commit started together in one worktree run one after the other; a held `index.lock` released within the bound lets the commit land; `fabricengine.RecordsRepoRoot` returns the one repository; a transition made while the worktree holds an unfinished code merge commits and pushes its records before the run acts past it, and concluding that merge, `git merge --abort` and `lym fabric merge --abort` each keep the records commit; a transition made during a cherry-pick, or during a merge fabric did not start whose index holds a records path unlike HEAD's, neither skips its commit and push nor lets the run act past it until the operation ends and the commit lands; a transition after a fabric merge crashed before D5 step 3 commits the status file it wrote, aborts that merge and pushes, leaving the status file in HEAD, the worktree and origin and no source records in any of them; `lym fabric merge --abort` of such a merge keeps an uncommitted receiving-side records edit in the commit it makes.
- D5: a local merge and a squash merge of a task into its parent, with uncommitted parent records and conflicting records edits on both sides, leave the parent's records equal to its own pre-merge records in index and worktree, with no task-only records file; a merge-in from the parent keeps the task's records; a merge whose source is a fast-forward of the receiver makes a merge commit carrying the merge trailer; an anchor-change commit merged into the parent changes no parent code path; a merge whose source edited `.lym-anchor` to another anchor refuses, naming the marker change; a receiving-side records write between step 2 and step 3 on a path the source changed makes the merge refuse; a merge verb run over a parked merge past step 3 refuses as already in progress and leaves its resolved conflicts in place; `lym fabric merge --continue` on a merge stopped before step 3 refuses, and Finalize over a parent pair holding one names only `lym fabric merge --abort` as its way forward.
- D6: a pair push pushes the branch once; a refused push leaves one failed-push record read as owed; after a takeover in a one-repository hub, origin's branch keeps every transition commit; teardown pushes the archive tag before deleting the branch on origin, deletes the view ref, and succeeds when origin already lost the view ref.
- D7: a pull over a diverged task branch with local records-only commits replays them with each trailer that named a replayed commit rewritten to the fork point; with a code commit among origin's new commits it replays them all the same, and no trailer names that commit; with a local code commit it rewrites nothing and stops naming it; a commit changing code paths whose message carries fabric's merge trailer is neither replayed nor admitted by the prime guard; a prime records push with an operator commit unpushed, including one fast-forwarded from a branch origin holds, pushes nothing and leaves an `operator_unpushed` failed push naming that commit and listing the prime runs, its notice is sent, and a run in the prime with that push owed does not act past its transition until the push lands; a prime records push refused as a protected branch leaves a `protected` failed push, and the run completes that transition and acts on with its records unpushed; a records commit in a prime switched off its recorded branch refuses, naming both branches, and leaves the records on disk; a records commit in a prime whose `lym.primeBranch` (new) key is absent refuses, naming the key; a fabric merge on the prime's branch made before the prime's `.lym` was removed is still admitted by the prime guard.
- D11: a board write in a one-repository hub commits and pushes on `lym-bolt` and changes no code branch.
- D12: `hubforge` builds a one-repository hub that the D1 to D11 scenarios run on.
- Review: D2: `docs/getting-started.md` and `README.md` describe cloning a one-repository hub, what it puts on code branches, the `--single-repo` hazard on a two-repository hub's code repository, and the hazards of switching, resetting or stashing in the prime.

## PATTERN changes

- `PATTERN-single-repo-hub` — adds: Running a hub as one repository: records on each code branch, the board on `lym-bolt`; merges, pull request views and code identity strip the records paths.
- `PATTERN-run-fsm` — changes: a transition commits and pushes before the run acts, except a prime records push failing as `protected`, which completes the transition with its records unpushed; an `operator_unpushed` push is owed.
- `PATTERN-cwd-resolution` — changes: the overview line reads the anchor from the worktree's own `.lym-anchor` at its records root, the weft root in a two-repository hub and the worktree root in a one-repository hub.
