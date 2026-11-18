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
  then creates `lym-bolt` as an orphan branch seeded as a fresh weft bind is today, the shortname included, and pushes both.

Bound: `--single-repo` creates only;
it refuses when origin already holds `lym-bolt`, holds a weft binding (`.lym-warp`), or a second URL is given.
It cannot recognise the code repository of an existing two-repository hub, whose origin holds neither;
run there, it starts a separate one-repository hub over the same code and changes nothing in the other hub.
The docs say so.
A single URL whose origin holds neither `lym-bolt` nor `--single-repo` keeps today's meaning: it is a weft URL.
`docs/getting-started.md` and `README.md` describe cloning a one-repository hub and what it puts on code branches.

### D3 — Worktree layout and the paths that no longer run

In a one-repository hub:

- `_lym` and `.lym` are real directories at the worktree's anchor; `.lym-anchor` sits at the root, tracked on the branch.
- `Add` creates the task worktree on its branch and writes the origin record and anchor marker as a records commit on it;
  no sibling worktree or second branch exists.
- Junction wiring, junction repair (the junction-missing cause in `internal/fabricengine/reconcile.go`), `.lym` content adoption and unwire do not run.
- The shared `.git/info/exclude` gets `.lym`, the `.scratch` link and heddle's rendered files, through fabric's exclude mutator as today;
  it never gets `_lym`, `.lym-anchor`, an `_lym` line per anchor, or the weft-side artifact lines `internal/fabricengine/recordsgit.go` seeds, which would hide code paths.
- `.scratch` links to the worktree's own `.lym/scratch` through `internal/fslink`.
- The hub workspace file lists no sibling folders.
- `lymcwd.Resolve` reads the anchor from the worktree's own marker; `ResolveWithin` accepts a cwd at or below the anchor only.

### D4 — Records are committed on the code branch, beside agents' work

Fabric's records commit (`lym fabric commit`, `sync`, the transition commit, `webstercli`'s sync) commits on the worktree's own branch:

- It is path-limited, as `git commit -- <paths>` is, through `fabricengine.ScopedPathspec`, so code an agent has staged is never swept in.
- Fabric's records and code commits in one worktree take one write lock.
- A commit that meets a held `index.lock` retries for a bounded time, a named constant, then fails as a commit failure does today.
- Its `Warp-SHA` trailer names HEAD before the commit; `Run-Transition` stays.
- `fabricengine.RecordsWorktree` and `fabricengine.RecordsRepoRoot` return the worktree and its repository,
  so weft-side readers outside fabric (`weavercli`, `battencli`'s catch-up, `reportcli`, `reportengine`) read the same repository unchanged.
- Archive and owner archive tags live in that repository.

Bound: an agent's own `git add -A` can put records paths into a code commit;
D5, D9 and D10 strip records by path, never by commit, so that is tolerated.

### D5 — Every merge keeps the receiving side's records

In a one-repository hub, every fabric verb that brings one branch's commits into another
(`fabricengine.Fabric.Merge` with or without squash, a merge-in from the parent into a task, a beam into its parent):

1. commits the receiving worktree's uncommitted records (D4);
2. merges without committing;
3. sets the records paths in both index and worktree to the receiving side's HEAD:
   restores what the source changed and deletes what only the source holds, each side's paths read from its own anchor marker;
4. hands the remaining code conflicts to today's handling, then commits with fabric's merge trailer, a named constant.

Fabric's own dirtiness guards (the merge verbs', `fabricengine.Fabric.FastForwardPair`'s) commit the worktree's records first in the same way and then judge code paths only.
A fast-forward of one branch onto another branch's commits is never made, since it would carry that branch's records; the verb makes a merge commit.
A fast-forward to the branch's own origin tip is unchanged.
An anchor change is a records commit that moves `_lym` and rewrites the marker; the anchor-change stop holds as today.

Bound: a records write that lands between step 1 and step 2 on a file the source changed makes git refuse the merge, which reads as today's dirty refusal.

Rationale: keeps `PATTERN-lym-files`' rule that records are per-branch and never a merge participant.

### D6 — Push, hand-over and teardown

- A pair push pushes the one branch once, under one push lock, so "records first, code only once it landed" holds trivially.
  The slug's teardown lock, the retry, the owed-push hold and the failed-push record with its creel notice stay;
  the failed-push record names the branch once, as its records side, so `PushOwed` and the act rule read it as today.
- A rejected push reads the run record at origin's fetched tip and hands over on another machine's takeover, as today.
- `fabricengine.OriginCodeTip` reads the trailer's commit, an ancestor of the pushed tip.
- Teardown tags the branch tip `archive/<slug>/<tip>`, pushes the tag, then deletes the branch locally and on origin, in today's order and locks;
  it deletes the D8 view ref too, and a view ref already gone on origin is no failure.

### D7 — Replay rewrites only records commits; the prime's records push is guarded

In a one-repository hub, every replay of local commits onto origin's tip (pull, origin reconcile, rejected-push recovery, `internal/fabricengine/replay.go`), on any branch:

- replays only records-only commits fabric made, rewriting their `Warp-SHA` trailer to the new parent, and sets aside ended-run commits only among those;
- when any other local commit is unpushed and origin diverged, rewrites nothing:
  a rejected push still reads a takeover as today, and otherwise the push or pull stops as owed, naming the commits.

A records push of the prime's branch, which is also the operator's code branch, pushes only when every commit origin lacks is a records-only commit or carries fabric's merge trailer.
Otherwise it is owed: its failed-push notice names the operator's unpushed commits and `git push` as the way forward,
and a run in the prime waits at its transition until the operator pushes.

Bound: fabric rewrites only its own unpushed records-only commits and never a code commit.
The guard binds records pushes; a landing's push of its parent carries the operator's unpushed commits on that branch, as it does today in both layouts.

### D8 — A pull request's head is a records-free view

Fabric gains one general verb that publishes a view of a branch against its parent and returns the ref a pull request uses as its head.

- Two repositories: it returns the branch and pushes nothing.
- One repository: it writes one commit whose parent is the merge base of branch and parent,
  whose tree is the branch's tree with the records paths taken from the merge base,
  and force-pushes it to `lym-view/<branch>` under the slug's teardown lock, shared as for a pair push.
  The parent is the branch's `parent_branch` in its fabric origin record.
- PR-Review, Finalize's pull-request path (`internal/landingshed/finalize_pr.go`) and `lym beam pr` publish before opening, updating or merging a pull request, and open it on the returned ref.
  Finalize's head check compares the pull request's head with the view tip it just published from the task tip.
- Fabric's push of a branch that has a local view ref refreshes the view; a failed view push leaves a failed-push record that holds no run, and the publish before a merge retries it.
- After GitHub merges a view, Finalize makes no local merge of the task:
  it fast-forwards the local parent to origin's parent tip, and the landed judgment is D9's held-work check.

Bound: the verb force-pushes only `lym-view/` refs, never a code or Bolt branch;
review comments on a refreshed view read as outdated; GitHub calls stay with `internal/githubclient`'s callers.

### D9 — Code identity ignores the records paths, inside fabric and out

Every judgment of code by tree, diff, dirtiness, changed files or held work excludes the records paths (`<anchor>/_lym`, `.lym-anchor`),
through one helper built on `lymdirs`, in a leaf package its readers may import.
It applies in both layouts; in a two-repository hub the paths are absent from the code tree, so nothing changes there.

- Tree identity: `verifytree.Verify`'s HEAD tree and the gate green record's (`internal/gatecli/green.go`) use the stripped tree.
- Dirtiness and changed files: `verifytree.DirtyPaths`, gate scope, weaver verify, drift, the landed size and the lint diffs leave records out.
- Held work: `fabricengine.Fabric.HoldsWork`, `fabricengine.ParentHoldsTaskWork`, `fabricengine.OriginParentHoldsTaskWork` and the landing and catch-up checks built on them
  read "the parent holds the task's work" as "merging the task's code into the parent changes no code".
- SHA readers: a check that HEAD is unchanged, meaning no code changed, becomes a code-tree check;
  a check that HEAD moved, meaning code was committed (webster's card checks in `internal/websterengine/gitwrap.go`), requires the code tree to change.
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

## Constraints

- The operator ruled that `_lym` is never in the exclude (board entry, 2026-11-18), and lym never force-adds (`PATTERN-gitrepo-client-boundary`).
- `PATTERN-fabric-boundary`: every git write on the one repository goes through `internal/fabricengine`; D8's verb is a general seam that knows nothing of pull requests.
- `PATTERN-gitrepo-client-boundary`: D8's force-push and every fetch and push go through `gitrepo` under the remote-refs lock.
- `PATTERN-lym-files`: `lymdirs` stays the only declarer of `_lym` and `.lym`.
- `PATTERN-cwd-resolution`: only `lymcwd` resolves the anchor; only fabric writes the marker or reads it from git.
- `PATTERN-code-conventions`: `lym-bolt`, `lym-view/`, the merge trailer and the `index.lock` retry bound are named constants.
- The Webster card that adds `PATTERN-single-repo-hub` also adds a one-repository paragraph to the backgrounds of `PATTERN-fabric-boundary` (excludes, Bolt `.gitignore`, teardown),
  `PATTERN-lym-files`, `PATTERN-cwd-resolution` (marker at the worktree root) and `PATTERN-pair-remote` (one push, guarded prime push); their overview lines stay.

## Auto-mode assumptions

- D4: records on the code branch, auto-picked.
- D1: the layout read from the tracked marker, auto-picked.
- D8: pull requests on a records-free view, auto-picked.
- Scope Out: conversion excluded and the mode built now, auto-picked.

## Open risks

- Board note `clean-room-module-rewrite`, and `design-simplify-callers`: the entry says the operator wants this mode taken in if fabricengine is rewritten, and both notes aim to shrink the package this record grows.
  This record builds the mode into the current package now. Whether the parent wants that, or a spec for the rewrite only, is the parent's call.
- D7 puts records commits on the operator's code branch in the prime; an operator who rebases that branch rewrites fabric's records commits too.

## Acceptance criteria

- D1: a worktree with `.lym-anchor` at its root reads as one repository, with `_bolt` missing too; a two-repository worktree with its weft sibling missing still reads as two.
- D2: clone of a URL whose origin holds `lym-bolt` makes a one-repository hub; `--single-repo` on a fresh origin commits the marker on the prime's branch and creates and pushes `lym-bolt`; `--single-repo` refuses an origin holding `lym-bolt` or a weft binding, and a second URL.
- D3: `Add` in a one-repository hub leaves `_lym` and `.lym` as real directories, `.lym-anchor` tracked at the root, `.lym` excluded, no `_lym` or weft-artifact exclude line, and no sibling worktree; `lym fabric reconcile` reports no junction cause; `lymcwd.Resolve` returns the anchor from the marker.
- D4: a records commit with code staged by another process commits only records paths, leaves the code staged, and names the prior HEAD in `Warp-SHA`; a held `index.lock` released within the bound lets the commit land; `fabricengine.RecordsRepoRoot` returns the one repository.
- D5: a local merge and a squash merge of a task into its parent, with uncommitted parent records and conflicting records edits on both sides, leave the parent's records equal to its own pre-merge records in index and worktree, with no task-only records file; a merge-in from the parent keeps the task's records; a merge whose source is a fast-forward of the receiver makes a merge commit carrying the merge trailer.
- D6: a pair push pushes the branch once; a refused push leaves one failed-push record read as owed; teardown pushes the archive tag before deleting the branch on origin, deletes the view ref, and succeeds when origin already lost the view ref.
- D7: a pull over a diverged task branch with local records-only commits replays them with rewritten trailers; with a local code commit it rewrites nothing and stops naming it; a prime records push with an operator commit unpushed pushes nothing and leaves an owed push naming that commit.
- D8: the view's three-way merge into the parent changes the parent's code as the branch does and leaves its records unchanged; a push after a new commit refreshes the view; after a GitHub merge of the view, Finalize fast-forwards the local parent, makes no merge commit, and reads the task as landed; in a two-repository hub the verb returns the branch and pushes nothing.
- D9: a records-only commit leaves the verify tree identity and the gate green record valid; uncommitted records edits do not make `verifytree.DirtyPaths` report dirt; changed-file sets omit records paths; `fabricengine.Fabric.HoldsWork` reads true for a parent that merged the task's code with D5's strip; a webster card check does not accept a records-only commit as code work.
- D10: a webster write under `_lym` gets the same audit verdict in both layouts.
- D11: a board write in a one-repository hub commits and pushes on `lym-bolt` and changes no code branch.
- D12: `hubforge` builds a one-repository hub that the D1 to D11 scenarios run on.
- Review: D2: `docs/getting-started.md` and `README.md` describe cloning a one-repository hub, what it puts on code branches, and the `--single-repo` hazard on a two-repository hub's code repository.

## PATTERN changes

- `PATTERN-single-repo-hub` — adds: Running a hub as one repository: records on each code branch, the board on `lym-bolt`; merges, pull request views and code identity strip the records paths.

## Notes for the plan writer

- `internal/fabricengine/merge.go`'s package comment says the weft is not a merge participant; D5 is how that stays true with one repository.
- The fabric-internal held-work check is in `internal/fabricengine/parenttips.go`; it compares merged trees, so stripping both trees is the change.
