# Decision record — finalize-dirty-names-paths

## Goal

When Finalize's parent-side merge refuses on a dirty parent checkout, the stuck reason names the checkout it checked and the dirty paths.
No automatic goto re-steps Finalize into the same refusal, and lyx's own run records never count as that dirt.

## Scope

In:

- Finalize's stuck reason for the parent-side dirty-worktree refusal names the parent's code checkout and lists its tracked dirty paths.
- `fabricengine`'s merge dirty guard hands the checkout and the paths it decided on to its caller on the typed guard error, outside `Error()`.
- The loom and darn goto guard refuses a goto that would lead straight back into Finalize while the parent's code checkout still has tracked dirt.
- The halt note's `parent fabric status:` section and the loom seam that fills it are removed, since the stuck reason now carries the list from the guard's own read.

Out:

- Any change to which checkout the merge guard checks: it stays the parent's code checkout alone.
- The wording of `MergeGuardError.Error()`, and every surface other than Finalize's parent-side arm that shows it: the fabric CLI's merge envelope, and the task-side `MergeIn` guard error that the merge-in resolver wraps, which keep the bare text.
  The old halt-note section listed the parent's paths for those halts, the wrong tree for a task-side refusal, so D3 loses nothing correct there.
- Cleaning, committing or stashing the parent's dirt automatically.
- The driver's rule table and the tackler prompt.

## Decisions

### D1 — Run records already never trip the merge guard; no exclusion is added

Decision: no code change for the issue's second point.
The parent-side merge guard (`pairDirtyReason`) reads only the parent's code checkout.
In a pair the code checkout's `_lyx` is a directory link into the records checkout, so `_lyx/shed/**` never shows in the code checkout's `git status`.
Finalize's step 2b (`commitParentRecords`) also commits the parent's run records before the merge, and again before the merge-in-required retry.
An acceptance criterion pins the behaviour so it cannot regress silently.

Rationale: the reported symptom is a misleading report, not a wrong guard; D3 removes the report that misled.

### D2 — The guard error carries its checkout and dirty paths; Finalize's stuck reason renders them

Decision: the dirty guard reads the code checkout's tracked dirty paths once and decides on that list: dirty means the list is not empty.
`MergeGuardError` gains fields holding the checked code checkout's path and those paths, worktree-relative and sorted, filled whenever the error carries the dirty-worktree reason, at every site that calls the dirty guard (`Merge`, `MergeIn`, the under-lock re-check).
`Error()` stays the fixed, path-free string the closed reason set requires.
Finalize's parent-side dirty arm (steps 6-7) builds its stuck reason from the guard error's text, the checkout path and the paths, followed by the way forward.
The way forward reads: commit or discard them in the parent pair, for example with `lyx fabric commit` there, then `lyx loom goto --to Finalize` and `lyx loom resume` in the task worktree.
The list holds at most 20 paths; a longer one ends in `and N more`.
One function renders the path list and the way forward for both D2 and D4, so the stuck reason and the goto refusal never drift apart.

Rationale: the issue's first point asks for the tree and the paths.
The package's side-free rule keeps paths out of `Error()`, and a typed field is the package's existing way to carry detail (`ErrMergeInRequired.Source`).
Finalize holds the parent only as a merge handle with no path, so the checkout path travels on the error too.
Deciding on the list itself means the error never says dirty with an empty list.
The cap bounds a reason that lands in the status file, the halt note and the orch's notice.
The way forward moves the run onto Finalize with a guarded goto, so a resume steps Finalize directly instead of spawning a tackler for a blocked stop.

### D3 — The halt note drops its parent fabric status section

Decision: `loomcli` no longer reads the parent pair's dirty paths for a halt note, and the `parentDirtyPaths` seam, `parentDirtyPathsFromFabric` and `renderParentFabricStatus` go.
`haltnote.Note` loses `ParentStatus`, and `Render` writes no `parent fabric status:` section.
`Parse` still recognizes the `parent fabric status:` header and skips the lines under it, so a note an older binary wrote parses as before.
`fabricengine.IsWorktreeDirtyReason` goes with its one production caller.

Rationale: after D2 the halt note's fenced reason holds the exact list the guard decided on.
The separate section was a second, later read across both checkouts, and it is what showed the run record as dirt.

### D4 — A goto back into Finalize is refused while the parent's code checkout is dirty

Decision: the loom and darn recipes' goto guard (built in `loomcli`, today `settledGuard`) gains an arm for two kinds of target: the Finalize row, identified by its engine rather than its name, and every row whose `OnDone` routes to it.
The second kind covers the tackler's routed-from row: a Finalize halt's last history entry is the row that routed into Finalize, and a resume there would route straight back.
The arm opens the parent pair the run's origin record names and reads its code checkout's tracked dirty paths through the same `fabricengine` read D2's guard decides on, never through `Fabric.DirtyPaths`, which also lists the records checkout.
It refuses when the list is not empty, with D2's rendering of the checkout, the paths and the way forward.
Every goto passes the guard (tackler and arbiter verdicts, their consume, an orch `lyx loom goto`), so a resume naming either kind of target hands back with the refusal text and no step runs.

Bound: the arm acts only on those targets and reads only the parent's code checkout; every other target goes to the existing settled-segment arm unchanged.
It is advisory, deciding on the dirt present at goto time: Finalize's own merge guard stays the authority, and dirt cleaned during Finalize's earlier steps or added after the goto is decided there.
The arm admits the goto and logs a warning when the parent pair cannot be opened or read, since Finalize's own steps then report the real problem.
It also admits when a fabric merge record exists on the parent pair, since that dirt is another run's merge in flight: Finalize's own guard then refuses with `merge already in progress`, and the refusal never tells the operator to discard another run's staged merge.

Rationale: the issue's third point.
A re-step changes nothing the merge guard reads, so it can only produce a second halt; the goto guard is the seam every goto passes.

## Constraints

- `MergeGuardError.Error()` stays fixed and path-free, per the closed reason set's rule in `internal/fabricengine/mergeerrors.go`; detail travels only in typed fields.
- Git on the parent pair runs only through `fabricengine` (`PATTERN-fabric-git`); the goto guard reads through a `fabricengine` method, never raw git.
- The goto guard only decides: it moves no run and writes no file (`shedengine.GotoGuard`'s contract).
- Every refusal and stuck reason names its way forward, and a test reaches it (`PATTERN-refusal-way-forward`); the way forward names no command agents' settings deny (`PATTERN-no-denied-recovery`).
- Halt notes stay tolerantly parsed (`PATTERN-tolerant-records`, `PATTERN-sole-parsers`): an older note still parses.

## Auto-mode assumptions

- A dirty parent is rare and small, so a 20-path cap loses nothing an operator needs in practice.
- `lyx loom goto --to Finalize` then `lyx loom resume` steps Finalize on a run halted there, as the settled-segment refusal's way forward already relies on for other rows.

## Open risks

- The tackler still spawns once at the first blocked Finalize stop before its resume is refused; that costs one tackler session per dirty-parent halt.
- The goto guard reads the parent's state at goto time; dirt that appears between the goto and the re-step still fails Finalize, and that re-step writes its halt note as today.

## Acceptance criteria

- Finalize, with a parent whose code checkout has tracked dirty paths, returns Stuck with a reason holding the guard error text, the parent's code checkout path, every dirty path worktree-relative, and the way forward.
- More than 20 dirty paths render as the first 20 sorted paths and `and N more`.
- A guard error carrying the dirty-worktree reason always carries a non-empty path list, from `Merge`, `MergeIn` and the under-lock re-check alike.
- A parent whose only uncommitted tracked path is a run record under `_lyx/shed/` in its records checkout lets Finalize's parent-side merge land (an existing test or a new table row).
- A goto onto Finalize, or onto a row whose `OnDone` routes to Finalize, is refused with the paths and the way forward while the parent's code checkout is dirty, and admitted once it is clean.
- A goto onto Finalize is admitted when the parent pair cannot be opened, and when a fabric merge record exists on the parent pair.
- A goto onto any other row behaves exactly as before, settled-segment refusals included.
- A tackler resume naming Finalize, or its routed-from row, under a dirty parent hands back with the refusal text and writes no second halt note.
- A halt note renders no `parent fabric status:` section, and a note holding one, as an older binary wrote it, parses with every other field intact.

## Notes for the plan writer

- `trackedDirtyPaths` already yields the code checkout's paths in the `-z` form that keeps spaces verbatim; it returns absolute paths, and D2 wants them relative to the checkout.
- `fabricengine.OpenParent` plus the run's recorded parent branch is how `loomcli/halt.go` reaches the parent pair today; D4 reuses that opening, not its `DirtyPaths` call.
