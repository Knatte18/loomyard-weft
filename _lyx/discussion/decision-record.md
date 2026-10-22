# Decision record — finalize-dirty-names-paths

## Goal

When Finalize's parent-side merge refuses on a dirty parent checkout, the stuck reason names the checkout it checked and the dirty paths.
No automatic goto, and no driver starting at a blocked Finalize stop, re-steps Finalize into the same refusal, and lyx's own run records never count as that dirt.

## Scope

In:

- Finalize's stuck reason for the parent-side dirty-worktree refusal names the parent's code checkout and lists its tracked dirty paths.
- `fabricengine`'s merge dirty guard hands the checkout and the paths it decided on to its caller on the typed guard error, outside `Error()`.
- The loom and darn goto guard refuses a goto onto Finalize while the parent's code checkout still has tracked dirt.
- A Go driver that starts at a blocked Finalize stop checks the parent's code checkout before its first step.
- The halt note's `parent fabric status:` section and the loom seam that fills it are removed, since the stuck reason now carries the list from the guard's own read.
- `contracts/specs/refusal-spec.md`'s way-forward list admits plain git in a checkout the message names, for the operator's own work (D2).

Out:

- Any change to which checkout the merge guard checks: it stays the parent's code checkout alone.
- The wording of `MergeGuardError.Error()`, and every surface other than Finalize's parent-side arm that shows it: the fabric CLI's merge envelope, and the task-side `MergeIn` guard error that the merge-in resolver wraps, which keep the bare text.
- Cleaning, committing or stashing the parent's dirt automatically.
- The driver's rule table and the tackler prompt.

## Decisions

### D1 — Run records already never trip the merge guard; no exclusion is added

Decision: no code change for the issue's second point.
The parent-side merge guard (`pairDirtyReason`) reads only the parent's code checkout, in its tracked-only scope.
In a pair the code checkout's `_lyx` is a directory link into the records checkout, which `fabricengine` creates through `internal/fslink` (`internal/fabricengine/junction.go`), and `wireJunctionSet` there appends the link's name to the code repository's `.git/info/exclude` (`/_lyx` in this hub).
So no file under `_lyx` is tracked in the code checkout, and the guard never lists one, on any platform.
Finalize's step 2b (`commitParentRecords`) also commits the parent's run records before the merge, and again before the merge-in-required retry.
An acceptance criterion pins the behaviour so it cannot regress silently.

Rationale: the reported symptom is a misleading report, not a wrong guard; D3 removes the report that misled.

### D2 — The guard error carries its checkout and dirty paths; Finalize's stuck reason renders them

Decision: the dirty guard reads the code checkout's tracked dirty paths once and decides on that list: dirty means the list is not empty.
`MergeGuardError` gains fields holding the checked code checkout's path and those paths, worktree-relative and sorted, filled whenever the error carries the dirty-worktree reason, at every site that calls the dirty guard (`Merge`, `MergeIn`, the under-lock re-check).
`Error()` stays the fixed, path-free string the closed reason set requires.

Finalize's parent-side dirty arm (steps 6-7) builds its stuck reason from the guard error's text, the checkout path and the paths, followed by the way forward, when the guard error carries the dirty-worktree reason and not `merge already in progress`.
When it carries both, Finalize's stuck reason is the error's own text with no path list, followed by the parent's code checkout path from the guard error's checkout field, so the plain git below has a named checkout, and the parked merge Finalize reads from the parent pair's merge record (`fabricengine.MidMerge`), its verb and its source, as the task-side parked-merge reason in `internal/landingshed/premerge.go` names them.
Its way forward reads: the party that started that merge, or the orch or the operator when that party is gone, concludes it with `lyx fabric merge --continue` or discards it with `lyx fabric merge --abort` in the parent pair, then `lyx loom goto --to Finalize` and `lyx loom resume` in the task worktree; `--abort` hard-resets the parent's code checkout to the merge's start, so the operator first commits or sets aside with plain git any change of their own in `<checkout>`; a session lyx refuses the verb from reports status: FAILED and the orch runs it.
`MergeAbort` (`internal/fabricengine/mergelifecycle.go`) resets through `resetMergeSides` with force, discarding every tracked change in the code checkout, and refuses with `merge conclude already landed` when a side's conclude may have landed, where `--continue` is the recovery.
When the record cannot be read, the reason names no verb and source and keeps the same way forward.
When the read reports no parked merge (the record concluded or removed between the guard and the read), `--continue` and `--abort` would refuse with no merge in progress (`MergeContinue` and `MergeAbort` in `internal/fabricengine/mergelifecycle.go`), so the reason names no verb and source, and its way forward is `lyx loom goto --to Finalize` and `lyx loom resume` in the task worktree alone: D4's guard then refuses with the dirt-only form while dirt remains.
`Merge` aggregates every failed precondition before refusing (`internal/fabricengine/merge.go`), and the dirt may then be another run's merge in flight, which no way forward may tell anyone to commit or discard.

When the guard error carries the dirty-worktree reason with any other reason but `merge already in progress`, such as `branch not synced to upstream` or `checkout is not on a branch`, the stuck reason keeps the error's full text, which names every reason, and the path list.
Its way forward names one step per reason, in an order set per reason, then the goto and the resume.
For `branch not synced to upstream`, which the guard raises only on a genuine divergence, the dirt step comes first in the dirt-only wording below, then the sync step in Merge-In's wording for the diverged state (`ParentDiverged` in `syncParent`, `internal/landingshed/mergein.go`): push or reconcile the parent branch in its own pair.
For `checkout is not on a branch` the checkout step comes first, since a commit on a detached HEAD lands where no ref reaches and the next checkout discards it (`detachedHeadReason` in `internal/fabricengine/mergeguards.go`).
`syncParent` words no detached state, so this step is new text written for this reason: check out the parent branch in `<checkout>` with plain git; when the checkout refuses to carry the changes, set them aside first with plain git and restore them on the branch after it.
The dirt step follows it and reads: commit them on the parent branch or discard them in `<checkout>` with plain git.
The two reasons never come together: a detached HEAD has no upstream, so the guard reports it synced (`upstreamSHAAt` in `internal/fabricengine/mergeguards.go`).
A source reason (`source branch not found`, `source branch is not fabric-managed`) concerns the task branch, not the parent, so the reason keeps it in the error's text and the way forward adds no step for it, as Finalize's generic merge-failure arm does today.

Each path is rendered Go-quoted, and the paths are joined by a comma and a space.
That form survives `stuckReason`, which collapses every line break in a reason to one space (`oneLine` in `internal/shedengine/run.go`), and keeps a path holding a space or a comma one item.
The list holds at most 20 paths; a longer one ends in `and N more`.

The way forward reads: commit or discard them in `<checkout>` with plain git, then `lyx loom goto --to Finalize` and `lyx loom resume` in the task worktree.
That plain git runs outside the task worktree, which `contracts/specs/refusal-spec.md`'s `What a way forward is` list does not admit today, so the task amends the list in the same commit:
item 3 widens from a plain `git` command in the task worktree to a plain `git` command in a checkout the message names, when the work there is the operator's and no `lyx` verb covers it.
The D2, D4 and D5 texts rely on that widened item; the plan writes the spec edit.
The dirt is the operator's, and only the operator clears it: the parent's code checkout is no task session's or orch's own worktree, and discarding throws away the operator's work.
The orch relays the reason to the operator and runs the goto and the resume once the operator has cleared it.
A commit there moves the local parent tip, so the re-stepped Finalize never lands it directly.
Finalize's step 1b2 (`holdParent`, then `parentGuard.moved` in `internal/landingshed/parentguard.go`) reads a local parent tip that differs from the merge record's merged tip and does not hold the task's work as moved, and Finalize's `on_reroute` is Merge-In (`contracts/recipes/loom-recipe.yaml`), so the goto and the resume reroute to Merge-In.
Merge-In's `syncParent` (`internal/landingshed/mergein.go`) then stops on a local parent ahead of its upstream, with its own way forward to push or reconcile the parent branch in its own pair, unless the push is skipped.
Once the operator has pushed, or under a skipped push at once, Merge-In merges the commit into the task branch, and it passes Merge-Bouncer's review perch, Publish and PR-Gate before Finalize lands.
So the operator commits there only what they would push themselves.
After a discard the parent tip is unchanged, and Finalize lands directly.
One function renders the checkout, the path list and the dirt-only way forward for D2, D4 and D5, so the stuck reason and the refusals never drift apart;
D4 and D5 read only the dirty paths, never a guard error, so they render that dirt-only form alone.

Settled by reading: `lyx loom goto --to Finalize` then `lyx loom resume` steps Finalize on a run halted there.
`gotoAdmitted` (`internal/shedengine/goto.go`) admits the current row of a blocked run, `Goto` leaves the run paused, and `lyx loom resume` (`internal/loomcli/resume.go`) spawns a Go driver for a halted run that has none, which steps it.

Rationale: the issue's first point asks for the tree and the paths.
The package's side-free rule keeps paths out of `Error()`, and a typed field is the package's existing way to carry detail (`ErrMergeInRequired.Source`).
Finalize holds the parent only as a merge handle with no path, so the checkout path travels on the error too.
Deciding on the list itself means the error never says dirty with an empty list.
The cap bounds a reason that lands in the status file, the halt note and the orch's notice.
The way forward names plain git, not `lyx fabric commit`: that verb stages only the records pathspec (`ScopedPathspec`) into the weft worktree (the `commit` verb in `internal/fabriccli/weft_verbs.go`, `Commit` in `internal/fabricengine/commit.go`), never the code checkout's dirt, and `sandboxedRoleRefusal` (`internal/fabriccli/roleguard.go`) refuses it from a task session.
The way forward goes through the goto because D4 guards it: while the parent is still dirty the goto refuses with the current list, before any driver is spawned.

### D3 — The halt note drops its parent fabric status section

Decision: `loomcli` no longer reads the parent pair's dirty paths for a halt note, and the `parentDirtyPaths` seam, `parentDirtyPathsFromFabric` and `renderParentFabricStatus` go.
`haltnote.Note` loses `ParentStatus`, and `Render` writes no `parent fabric status:` section.
`Parse` still recognizes the `parent fabric status:` header and skips the lines under it, so a note an older binary wrote parses as before.
`fabricengine.IsWorktreeDirtyReason` goes with its one production caller.

Rationale: after D2 the halt note's fenced reason holds the exact list the guard decided on.
The separate section was a second, later read across both checkouts, and it is what showed the run record as dirt.

### D4 — A goto onto Finalize is refused while the parent's code checkout is dirty

Decision: the loom and darn recipes' goto guard (built in `loomcli`, today `settledGuard`) gains an arm for the Finalize row, identified by its engine rather than its name.
The arm is its own function, which the goto guard composes with the settled-segment arm and D5 uses alone.
The arm opens the parent pair the run's origin record names and reads its code checkout's tracked dirty paths through the same `fabricengine` read D2's guard decides on, never through `Fabric.DirtyPaths`, which also lists the records checkout.
It refuses when the list is not empty, with D2's dirt-only rendering of the checkout, the paths and the way forward.
Every goto passes the guard (tackler and arbiter verdicts, their consume, an orch `lyx loom goto`), so a resume naming Finalize hands back with the refusal text and no step runs.

Finalize is the only target.
A Finalize Stuck with no `on_stuck` appends Finalize's own history entry (`appendHistory` in `internal/shedengine/run.go`), and the tackler's `gotoRows` (`internal/sheddriver/tackler.go`) adds the routed-from row only when it differs from the current row, so at a Finalize halt the tackler can name only Finalize.

Bound: the arm acts only on Finalize and reads only the parent's code checkout; every other target goes to the existing settled-segment arm unchanged.
Its refusal is the refusal spec's transient class (`contracts/specs/refusal-spec.md`): the dirt clears by a named action, the arm refuses without mutating state, and its text names the retry.
So it halts the goto, as `PATTERN-refusal-way-forward` allows a transient refusal to, rather than warning as a policy guard would.
It decides on the dirt present at goto time: Finalize's own merge guard stays the authority, and dirt cleaned during Finalize's earlier steps or added after the goto is decided there.
The arm admits the goto and logs a warning when the parent pair cannot be opened or read, since Finalize's own steps then report the real problem.
It also admits when a fabric merge record exists on the parent pair, since that dirt may be another run's merge in flight.
Finalize's own guard then refuses with both `merge already in progress` and `worktree dirty`, and D2 renders the error's own text with no path list and the merge-in-progress way forward, so no one is told to commit or discard the dirt of another run's staged merge with plain git.

Rationale: the issue's third point.
A re-step changes nothing the merge guard reads, so it can only produce a second halt; the goto guard is the seam every goto passes.

### D5 — A driver starting at a blocked Finalize stop checks the parent before it steps

Decision: a Go driver whose first status read finds the run blocked, with no open attempt to resume, runs a told check on the current row before its first step.
`loomcli` wires that check to D4's Finalize arm alone, never the settled-segment arm, so a blocked stop at any other row steps as today.
A refusal hands back with the refusal text and steps nothing; an admit, or a recipe that wires no check, steps as today.
This covers the hand-over successor and the driver a bare `lyx loom resume` spawns at a blocked Finalize stop.
The refusal is the same transient class as D4's.

Rationale: at a blocked stop `binaryEligible` is true, and `decideDrive` picks the hand-over before the tackler rule when the on-disk build changed (`internal/sheddriver/driverules.go`).
The successor finds no open attempt and calls `Step` (`run` and `handOver` in `internal/sheddriver/drive.go`), and `Step` re-calls `current_producer` on a blocked run (`internal/shedengine/run.go`), so Finalize would re-run into the same refusal and write a second halt note.
This hub deploys after every landing, so a changed build at a blocked stop is routine.
The check sits at the driver's start, not in the rule table, because the table's only re-step without a goto is the interrupted-step rule, never a blocked stop, so a driver's first step is where a blocked Finalize re-runs unguarded.

## Constraints

- `MergeGuardError.Error()` stays fixed and path-free, per the closed reason set's rule in `internal/fabricengine/mergeerrors.go`; detail travels only in typed fields.
- Git on the parent pair runs only through `fabricengine` (`PATTERN-fabric-git`); the goto guard and the driver's check read through a `fabricengine` method, never raw git.
- The goto guard only decides: it moves no run and writes no file (`shedengine.GotoGuard`'s contract).
- Every refusal and stuck reason names its way forward, and a test reaches it (`PATTERN-refusal-way-forward`); the way forward names no command agents' settings deny (`PATTERN-no-denied-recovery`).
  The plain git in the parent's code checkout holds only under D2's amendment of `contracts/specs/refusal-spec.md`'s way-forward list, made in the same commit.
- Halt notes stay tolerantly parsed (`PATTERN-tolerant-records`, `PATTERN-sole-parsers`): an older note still parses.

## Auto-mode assumptions

- A dirty parent is rare and small, so a 20-path cap loses nothing an operator needs in practice.

## Open risks

- The tackler still spawns once at the first blocked Finalize stop before its resume is refused; that costs one tackler session per dirty-parent halt.
- An orch goto onto a row that routes into Finalize, such as PR-Gate, is admitted, and that row's Done steps Finalize inside the driver's loop with no check, so a still-dirty parent writes a second halt note; the route starts only from a deliberate orch goto.
- The goto guard and the driver's check read the parent's state when they run; dirt that appears between then and the re-step still fails Finalize, and that re-step writes its halt note as today.

<!-- lyx:carry-over Discussion-Review -->
- Discussion-Review, round 3: converged; each finding below was fixed or disputed by the round's fixer, and no fresh reviewer has seen the result; the fixer report says which.
  Review: _lyx/reviews/discussion/round-3-review.md; fixer report: _lyx/reviews/discussion/round-3-fixer-report.md.
  - checkout-step-ordered-after-dirt-step (class design, severity MEDIUM, open in rounds 3)
  - merge-in-progress-arm-names-no-checkout (class consistency, severity MEDIUM, open in rounds 3)
<!-- /lyx:carry-over Discussion-Review -->

## Acceptance criteria

- Finalize, with a parent whose code checkout has tracked dirty paths, returns Stuck with a reason holding the guard error text, the parent's code checkout path, every dirty path up to D2's cap worktree-relative, Go-quoted and joined by a comma and a space, and the way forward.
- More than 20 dirty paths render as the first 20 sorted paths and `and N more`.
- Finalize, with a dirty parent and a fabric merge record on it, returns Stuck with the guard error's own text, naming both reasons, no path list, the parent's code checkout path, the parked merge's verb and source, and a way forward naming `lyx fabric merge --continue`, `lyx fabric merge --abort`, the warning that `--abort` resets the parent's code checkout, the role-guard clause, `lyx loom goto --to Finalize` and `lyx loom resume`.
  When the merge record cannot be read, the reason names no verb and source and keeps the same way forward;
  when the read reports no parked merge, the reason names no verb and source, and its way forward names only `lyx loom goto --to Finalize` and `lyx loom resume`.
- Finalize, with a dirty parent whose guard error also carries `branch not synced to upstream`, returns Stuck with a reason naming both reasons, the path list, and a way forward naming the dirt step, then the sync step, then the goto and the resume.
- Finalize, with a dirty parent whose guard error also carries `checkout is not on a branch`, returns Stuck with a reason naming both reasons, the path list, and a way forward naming the checkout step, then the dirt step, then the goto and the resume.
- After an operator commit in the parent's code checkout, `lyx loom goto --to Finalize` and `lyx loom resume` reroute to Merge-In with no dirty halt; after a discard, Finalize lands.
- `contracts/specs/refusal-spec.md`'s way-forward list admits a plain `git` command in a checkout the message names, when the work there is the operator's and no `lyx` verb covers it.
- A guard error carrying the dirty-worktree reason always carries a non-empty path list, from `Merge`, `MergeIn` and the under-lock re-check alike.
- A parent whose only uncommitted tracked path is a run record under `_lyx/shed/` in its records checkout lets Finalize's parent-side merge land (an existing test or a new table row).
- A goto onto Finalize is refused with the paths and the way forward while the parent's code checkout is dirty, and admitted once it is clean.
- A goto onto Finalize is admitted when the parent pair cannot be opened, and when a fabric merge record exists on the parent pair.
- A goto onto any other row behaves exactly as before, settled-segment refusals included.
- A tackler resume naming Finalize under a dirty parent hands back with the refusal text and writes no second halt note.
- A Go driver starting at a blocked Finalize stop under a dirty parent, whether a hand-over successor or one a bare `lyx loom resume` spawned, hands back with the refusal text, steps nothing and writes no second halt note; it steps Finalize when the parent pair cannot be opened or a fabric merge record exists on it, and at a blocked stop on any other row it steps as before.
- A halt note renders no `parent fabric status:` section, and a note holding one, as an older binary wrote it, parses with every other field intact.

## Notes for the plan writer

- `trackedDirtyPaths` already yields the code checkout's paths in the `-z` form that keeps spaces verbatim; it returns absolute paths, and D2 wants them relative to the checkout.
- D2's merge-in-progress arm reads the parked merge with `fabricengine.MidMerge` on the parent pair's location, never on the task worktree's.
- D4 reaches the parent pair through the `loomCLI.recordedParentBranch` seam and `fabricengine.OpenParent`, and reads only the code checkout.
