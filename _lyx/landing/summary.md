# Finalize names the dirty parent checkout and its paths

When Finalize's parent-side merge refused on a dirty parent, the stuck reason said only `worktree dirty`.
Now it names the parent's code checkout, lists its tracked dirty paths, and gives a way forward.
A goto onto Finalize, and a Go driver starting at a blocked Finalize stop, no longer re-step into the same refusal while the parent is still dirty.

Closes #776

## What changes

- `fabricengine`: the dirty merge guard reads the code checkout's tracked dirty paths once and decides on that list.
  `MergeGuardError` carries the checkout path and the sorted, worktree-relative paths in typed fields; `Error()` stays fixed and path-free.
  The error carries them from `Merge`, `MergeIn` and the under-lock re-check alike.
  `Fabric.CodeDirtyPaths` and `Fabric.MidMerge` are new read-only parent reads.
- `landingshed`: Finalize's dirty-parent arm builds its stuck reason through `DirtyParentRefusal`.
  Paths are Go-quoted, joined by a comma and a space, and capped at 20 with `and N more`.
  The way forward is per reason:
  - dirt only: commit or discard in the checkout with plain git, then `lyx loom goto --to Finalize` and `lyx loom resume`;
  - dirt with `branch not synced to upstream` or `checkout is not on a branch`: one step per reason in a fixed order, then the goto and the resume;
  - dirt with `merge already in progress`: the error's own text with no path list, the parked merge's verb and source, and `lyx fabric merge --continue` or `--abort`, since that dirt may be another run's merge in flight.
- `loomcli`: the goto guard gains a Finalize arm that refuses with the same dirt-only rendering while the parent's code checkout is dirty.
  A Go driver whose first status read finds a blocked run runs that arm alone before its first step.
  Both admit, with a logged warning where the parent cannot be read, when the parent pair cannot be opened or a fabric merge record exists on it.
- `haltnote`: the `parent fabric status:` section and the loom seam that filled it are removed, as the stuck reason now holds the guard's own list.
  `Parse` still skips that section, so older notes parse unchanged.
  `fabricengine.IsWorktreeDirtyReason` goes with its one caller, and the friction reflection stencil drops its line about the section.
- `contracts/specs/refusal-spec.md`: item 3 of the way-forward list now admits plain `git` in a checkout the message names, when the work there is the operator's and no `lyx` verb covers it.

## Design decisions

- The issue's second point needs no code change.
  In a pair the code checkout's `_lyx` is a directory link excluded through `.git/info/exclude`, so no run record is tracked there and the guard never lists one.
  The misleading listing came from the halt note's separate section, which this change removes.
  A test pins that a parent whose only uncommitted path is a run record in its records checkout still lets Finalize land.
- The way forward names plain git, not `lyx fabric commit`, which stages only the records pathspec and is refused from a task session.
- After an operator commit in the parent, the goto and resume reroute to Merge-In, not directly to a landing, because the local parent tip has moved.
  After a discard the tip is unchanged and Finalize lands.
- The goto guard only decides; it moves no run and writes no file.

## To check by hand

- The merge-in-progress way forward in Finalize's dirty-parent reason (`landingshed.DirtyParentRefusal`) was flagged as possibly departing from the decision's wording; compare its text with the intended verb, warning and role-guard clauses.
- The checkout step and the dirt step ordering for `checkout is not on a branch`, and the merge-in-progress arm naming the parent's code checkout, were flagged in design review; read both in `DirtyParentRefusal`.
- Not exercised against a real dirty parent end to end: an operator commit then `lyx loom goto --to Finalize` and `lyx loom resume`, and the hand-over successor driver at a blocked Finalize stop.
- Known gaps: the tackler still spawns once at the first blocked Finalize stop before its resume is refused; an orch goto onto a row that routes into Finalize, such as PR-Gate, is admitted and can write a second halt note; dirt that appears between the guard's read and the re-step still fails Finalize.
