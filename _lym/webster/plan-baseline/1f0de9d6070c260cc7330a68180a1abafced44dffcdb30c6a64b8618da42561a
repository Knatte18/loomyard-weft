---
format: 5
approved: true
language: go
---

# Plan: one-repository layout in fabric

Build the one-repository hub layout into the current `internal/fabricengine` (the decision record's D1 to D7, D11 and D12).
A worktree whose root carries `.lym-anchor` belongs to a one-repository hub: its records live under `<anchor>/_lym` on its own code branch, and the board lives on the orphan branch `lym-bolt`.
The cards read the layout in `lymcwd`, define the records paths and commit predicates, point fabric's records geometry at the worktree itself,
then take clone, Add, the records commit, the anchor change, merge, the mid-merge commit, the transition seam, push and teardown, replay and the prime's guarded push through that layout,
and close with the docs and the PATTERN changes. Two-repository hubs keep today's behavior throughout;
the transition seam's wait on a blocking git operation applies in a one-repository worktree only, and a two-repository worktree keeps its skip (card 11).

## Card Index

1 — lymcwd-one-repository — read the anchor from the worktree-root marker and export the one-repository predicate
2 — records-paths-and-commit-predicates — RecordsPathsAt, the records-only and fabric-merge predicates and the merge and anchor-change trailers
3 — one-repository-geometry — records worktree, repository, lock directory and branch name resolve to the worktree itself
4 — protected-push-class — move the protected-branch match into fabric and record a protected failed-push class
5 — one-repository-clone — clone by one URL or --single-repo, Bolt on lym-bolt, lym.primeBranch, junction-free wiring and the hubforge builder
6 — one-repository-add — Add a task worktree on its branch with no sibling, and retire the junction paths
7 — records-commit-on-code-branch — path-limited records commit with index.lock retry and the prime-branch refusal
8 — one-repository-anchor-change — the anchor change as one trailer-carrying records commit
9 — merge-keeps-receiving-records — every fabric merge restores the receiving side's records and commits with the merge trailer
10 — records-commit-during-merge — records commits around a merge, abort and continue of an unsettled merge, and MergeStateOperation
11 — transition-seam-waits — statuscommit waits on a blocking operation and completes on a protected-only push failure
12 — one-branch-push-and-teardown — push the one branch once, no owed code tip, teardown with the view ref
13 — records-only-replay — replay only records-only commits with fork-point trailers
14 — prime-records-push-guard — guard the prime's records push and owe it as operator_unpushed
15 — one-repository-docs — getting-started and README describe the one-repository clone and its hazards
16 — pattern-single-repo-hub — PATTERN-single-repo-hub, the run-fsm and cwd-resolution changes and the backgrounds

## Shared Decisions

### Decision: layout-read

- **Decision:** `lymcwd` reads the layout from the working tree through `lymcwd.OneRepository`, which leaves out a records sibling and the Bolt checkout by name.
  Fabric reads it from git through one helper, `oneRepositoryAt(path)` (card 3): false when the main worktree of the repository holding `path` is a records sibling, which is a two-repository records repository;
  otherwise true when `.lym-anchor` is committed at the HEAD of the path's own worktree, or, for the main worktree and the Bolt checkout, whose checked-out branch may lack it, when the repository's `lym-bolt` branch commits it.
  A worktree Add has not made yet is read at its start point's commit.
  No config key, flag or hub file records the layout (D1).
- **Rationale:** D1 has `lymcwd` read the marker file and fabric read it from the branch in git; in a two-repository hub the records sibling's and the Bolt's branches carry the marker too, so both reads leave them out.
- **Applies to:** all cards

### Decision: records-lock-in-gitdir

- **Decision:** A one-repository worktree's records write lock, push lock and their companions sit in `<gitdir>/lym-locks`, not in `.weft/`.
- **Rationale:** D3 bars the weft artifact exclude lines, which would hide code paths; the gitdir needs no exclude, as `PATTERN-lym-files` already places push locks in the git dir.
- **Applies to:** cards 3, 7, 10, 12

### Decision: outside-readers-unchanged

- **Decision:** `RecordsBranchName` and its callers outside fabric stay as they are; fabric's own sites take the layout-aware `recordsBranchIn`.
- **Rationale:** D4 has weft-side readers outside fabric read the one repository unchanged through `RecordsWorktree` and `RecordsRepoRoot`.
- **Applies to:** card 3

## PATTERN changes

- `PATTERN-single-repo-hub` — adds: Running a hub as one repository: records on each code branch, the board on `lym-bolt`; merges, pull request views and code identity strip the records paths.
- `PATTERN-run-fsm` — changes: a transition commits and pushes before the run acts, except a prime records push failing as `protected`, which completes the transition with its records unpushed; an `operator_unpushed` push is owed.
- `PATTERN-cwd-resolution` — changes: the overview line reads the anchor from the worktree's own `.lym-anchor` at its records root, the weft root in a two-repository hub and the worktree root in a one-repository hub.
