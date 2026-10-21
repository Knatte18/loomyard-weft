# Decision record — scratch-defeats-test-cache

## Goal

In a fabric worktree whose `.scratch` is wired as a link, creating, changing or removing files under `.scratch/` no longer changes any input `go test` keys its cache on, so source-scan tests that read the module root keep their cache hits while an agent writes scratch files.

The miss comes from the root listing alone: a test that opens a directory makes the cache key include the `Lstat` of every entry in it, and the `scankit` walk opens the module root, whose real `.scratch` entry changes modification time on every create or remove under it.

## Scope

In:

- Fabric wiring makes a warp worktree's root `.scratch` a directory link (`internal/fslink`) into the pair's weft-side `.lyx` directory, so the root listing holds a link whose own `Lstat` does not move when files under it change.
- Migration of an existing real `.scratch` directory, the hub prime's included, into the link target.
- The warp `.git/info/exclude` entry for scratch covering a link as well as a directory.
- `lyx fabric reconcile` converging the scratch link and exclude of every managed pair.
- `lyx fabric remove` unlinking the root `.scratch` link before it removes the warp worktree directory.
- Docs: `internal/fabricengine/doc.go` and `internal/cleanupcli/doc.go`.

Out:

- The path agents write to: it stays `.scratch/` at the worktree root, so the edit directive, stencils and skills are unchanged.
- `scankit` and the source-scan tests: they already skip `.scratch` as a walk target, and the root listing they open is a legitimate cache input that must stay hashed.
- Checkouts fabric never wires, such as a plain clone: they keep a real `.scratch`.
- Other root-level churn, such as a `go build` leaving `lyx` in the root or files created directly under `.claude/`.

## Decisions

### D1. `.scratch` is a fabric-wired directory link into the weft's `.lyx`

One fabricengine scratch step owns both the warp worktree's root `.scratch` link and its exclude entry (D4).
Fabric wiring runs it from `WireJunctionsWith` on the worktree add, checkout and prime clone-and-wire paths only, after the existing junction and exclude steps.
The step replaces `seedScratchExclude`'s call in `WireJunctionsWith`, `seedScratchExclude` is deleted, and `ExcludeAnchoredDir` stays for its other callers.
Reconcile's re-wire calls a `WireJunctionsWith` form that skips the step, and reconcile runs the step itself per D3.
The link is created through `fslink.CreateDirLink`, and its target is a `scratch` directory inside the pair's weft-side `.lyx` directory.
The target path is built from the pair's weft worktree root, the hub's `AnchorRel` and `lyxdirs.DotLyxDirName`, as `WarpJunctions` builds every weft-side target, never by resolving the warp `.lyx` link, so a broken warp `.lyx` link does not move it; the `scratch` leaf and `.scratch` link name stay fabric's own constants, as `scratchDirName` already is.

The step writes the exclude first (D4).
When that write fails, the step stops there: it creates no link, renames nothing and reports the exclude path, the error and the way forward: clear what blocks writing the exclude file, then rerun `lyx fabric reconcile`.
Only after the write succeeded, or found the pattern already present, does it classify the root `.scratch`, and only then touch the target, in this order:

1. Root `.scratch` absent: `MkdirAll` the target, then create the link.
2. Root `.scratch` a link whose literal target, read through `fslink.RawTarget`, equals the target path: `MkdirAll` the target, which heals a dangling link after a weft re-creation; nothing else changes.
   The comparison never resolves the link, so a dangling link to the target is this case and never a link to another target.
3. Root `.scratch` a real directory holding no tracked path, and the target absent: `os.Rename` the directory to the target, then create the link, so its content and every `.scratch/<name>` path survive.
   Whether a tracked path lies under the root `.scratch` is asked of git through `internal/fabricengine`'s `PathTracked`, which runs `git ls-files -- .scratch` in the warp worktree.
4. The leave cases: a link to another target, a real directory while the target already exists, a regular file, or a failed rename.
   The step leaves the entry untouched and reports the path, the shape and the way forward: move the content into the target and remove the root `.scratch`, then rerun `lyx fabric reconcile`.
   A root `.scratch` git tracks, or one holding tracked content, is also a leave case, whatever its shape and whether or not the target exists.
   Its report names that shape and the way forward: stop tracking `.scratch` on the warp branch, move the untracked content into the target and remove the root `.scratch`, then rerun `lyx fabric reconcile`.
5. A link creation refused after a successful rename, because something recreated `.scratch` in between.
   The moved content now lives at the target alone.
   The step leaves the recreated `.scratch` untouched and reports the target path where the moved content lives, and the way forward: move the recreated `.scratch`'s content into that target, remove the root `.scratch`, then rerun `lyx fabric reconcile`.
6. The target's `MkdirAll` fails in case 1 or 2, case 1's link creation is refused, or the tracked-path check fails.
   The step leaves the root `.scratch` as it found it and reports the path, the error and the way forward: clear what the error names, moving any content under a root `.scratch` into the target before removing it, then rerun `lyx fabric reconcile`.

Rationale: `os.ReadDir` entries report `Lstat` information, so the hashed entry becomes the link itself, whose modification time is fixed at creation.
Writes through the link change only the target, which lies outside the module root.
The weft `.lyx` is never tracked, and scratch now lives and dies with the pair's weft worktree, including a weft that reconcile recreates, while agents keep writing `.scratch/<name>` unchanged.
Wiring the link at worktree creation also removes the first `mkdir .scratch` from the root listing's history.

### D2. The scratch step never fails wiring and never removes anything

No outcome of the scratch step fails `WireJunctionsWith`, fails a reconcile pair or refuses a run.
The step removes no file, directory or link: it never re-points a wrong link and never clears a target, so `internal/fabricengine/destroy.go` is not involved.
Every link it creates, every rename and every exclude write is recorded on the verb's `*Mutations` record, per `PATTERN-mutation-record`.
A link is recorded as `link_created` and an exclude write as `file_written`.
A rename is recorded under a new kind, `path_renamed`, whose Target is the target path and whose Detail is the renamed root `.scratch`; it lands in the same commit as its recording site.
A leave case of D1 case 4, D1 case 5, D1 case 6 or a failed exclude write is reported in the pair's reconcile result detail under reconcile, and logged through `internal/logger` under add, checkout and clone, whose results carry no detail field.

Bound: a failed exclude write leaves the worktree as it was before the step.
A leave case of D1 case 4 keeps that worktree's cache misses and nothing worse, until the operator clears it as the report says.
D1 case 5 also leaves the moved content reachable only at the target, not through `.scratch/<name>`, until the operator clears it as the report says.
D1 case 6 keeps that worktree's cache misses;
after a refused case-1 link creation it also leaves an empty target, and the next pass classifies whatever then holds the root `.scratch`:
a real directory or a file is a case-4 leave until the operator clears it as the report says, a link to the target is case 2, and an absent root is case 1 again.
The rename never moves tracked content, so the step never leaves a tracked path deleted in git's view.
This covers an agent's `rm -rf .scratch` followed by `mkdir .scratch`: the target still exists, so reconcile reports the real directory each pass and never converges it on its own.

Rationale: scratch is agent convenience and wiring gates worktree creation, so a scratch problem must never block a task or delete content.

### D3. Reconcile converges managed pairs; health checks stay scratch-blind

`checkJunctionHealth`, `PairComplete` and `preflight`'s `Healthy` do not look at `.scratch`.
`repairPairWiring` runs the scratch step once itself, on its healthy and its re-wired branch alike, after the junction repair whether or not that repair failed.
The step returns its outcome to `repairPairWiring`, which adds a detail to the pair's result when the step changed or left something.
It never runs the step on a worktree reported as unmanaged, on a raw-adopted worktree, which never reaches `repairPairWiring`, or on a pair whose weft worktree root is missing at the time of the step; with no weft root, the step creates nothing.
`lyx fabric unwire` leaves the scratch link and its exclude in place, as it leaves the scratch exclude today; the weft it points into stays, so the link stays valid.

The stale-junction sweep, `applyStaleRemoval`, and `lyx fabric unwire` never claim the root `.scratch` link, so neither removes the link nor strips its `/.scratch` exclude line.
At a root anchor, where `scanOnDiskJunctionNames` returns the root link, each drops the scratch link name from the scanned names; at a subpath anchor the scan never sees the root link, so there is nothing to drop.
`lyx fabric remove` unlinks the worktree-root `.scratch` link itself, by its root path and independent of `scanOnDiskJunctionNames`, which reads `<worktree>/<AnchorRel>` and so never sees the root link on a subpath-anchored hub.
It does so through `destroy.go`'s `removeLink` gate, recorded as `link_removed`, before it removes the warp worktree directory, and only when the root `.scratch` is a link `linkIsFabricOwned` claims, the same ownership test the scan applies.
At a root anchor it drops the scratch name from the scanned names, so the link is unlinked once.

Bound: the step runs only under the wiring verbs of D1 and under `lyx fabric reconcile` on a pair whose weft exists; no status read, preflight or other verb touches `.scratch`, apart from `lyx fabric remove`'s unlink of the link.

Rationale: junction health gates runs through `PairComplete` and `preflight`, so a missing or real `.scratch` must not refuse a run.
Reconcile is the existing operator verb for converging old worktrees, so the prime and every pair wired by an older binary convert there: after this lands, the operator runs `lyx fabric reconcile` once, and the fabricengine doc says so.

### D4. The scratch exclude matches the link

The scratch step writes the anchored pattern `/.scratch`, with no trailing slash, unconditionally, and replaces a legacy `/.scratch/` line it finds, the way `seedGitExclude` replaces its legacy bare-name lines.
It runs no `git check-ignore` probe first; the write is a no-op only when `/.scratch` is already present and no legacy line remains.

Rationale: git does not follow a link, and a pattern with a trailing slash matches only a directory, so `/.scratch/` would leave the link untracked dirt in `git status`, in dirty-tree checks and in `git worktree remove`.
For the same reason a trailing-slash probe, as `ExcludeAnchoredDir` runs, answers ignored for a link it does not cover: this repo's tracked `.gitignore` line `**/.scratch/` makes `git check-ignore .scratch/` succeed while the link stays untracked.
`/.scratch` matches the link, a Windows junction and a real directory alike.
Because the scratch step owns the exclude, reconcile rewrites it on healthy pairs too, not only on pairs whose junctions it re-wires.

### D5. Cleanup follows the prime's link

`lyx cleanup`'s scratch segment keeps listing the prime's `.scratch` by that path; `os.ReadDir` follows the link to the target, so the segment judges and removes the target's direct children as before and leaves the link.
Its doc states that the prime's `.scratch` is a fabric link and the segment works on what it points at.

Rationale: the segment's contract is about the children of `.scratch/`, which keep their names under D1.

### D6. The Windows measurement goes onto `fabric-windows`

The run appends to the `fabric-windows` board entry's body, through `lyx board`, a check that a directory junction's own modification time stays fixed when files are created under its target, naming the scratch link as the case that relies on it.

Rationale: `fabric-windows` is where fabric's junction behaviour gets verified on a Windows host, and its body does not mention scratch today, so without this the risk has no owner.

## Constraints

- `PATTERN-fabric-git`, `PATTERN-mutation-record`, `PATTERN-fabric-destruction-chokepoint`: the scratch step lives in `internal/fabricengine`, records its mutations and removes nothing.
- `PATTERN-lyxdirs-single-declarer`: the target's `.lyx` segment comes from `lyxdirs.DotLyxDirName`.
- `PATTERN-durable-vs-ephemeral-state`: scratch lands under the never-tracked `.lyx`; no run needs it.
- `PATTERN-never-force-add`: the exclude stays in the warp repo's `.git/info/exclude`.
- `CLAUDE.md` filesystem links: the link is created only through `internal/fslink`.

## Auto-mode assumptions

- The fix belongs in fabric wiring, not in the gate or the scan kit.
- Migration of existing worktrees happens on `lyx fabric reconcile` and on re-wiring, run once by the operator after landing; no automatic trigger is added.

## Open risks

- On Windows, the design assumes a directory junction's own modification time does not move when files are created under its target; the board entry `fabric-windows` carries that check per D6.

<!-- lyx:carry-over Discussion-Review -->
- Discussion-Review, round 2: converged; each finding below was fixed or disputed by the round's fixer, and no fresh reviewer has seen the result; the fixer report says which.
  Review: _lyx/reviews/discussion/round-2-review.md; fixer report: _lyx/reviews/discussion/round-2-fixer-report.md.
  - case-3-moves-tracked-content (class design, severity MEDIUM, open in rounds 2)
  - remove-unlink-blind-on-subpath-anchor (class design, severity MEDIUM, open in rounds 2)
  - target-creation-and-case-1-refusal-unbounded (class design, severity MEDIUM, open in rounds 2)
<!-- /lyx:carry-over Discussion-Review -->

## Acceptance criteria

- Wiring a new pair leaves `.scratch` at the warp root as a link resolving to `scratch` under the weft-side `.lyx` directory.
- After a file is created under the wired `.scratch/`, the `Lstat` of the warp root's `.scratch` entry, modification time included, is unchanged.
- Re-wiring a pair with a correct link changes nothing and records nothing; a correct link whose target is missing gets its target back.
- Wiring a worktree whose `.scratch` is a real directory with files, and whose target is absent, moves the files to the target and leaves `.scratch/<name>` readable through the link.
- A conversion records the link as `link_created`, the rename as `path_renamed` and the exclude write as `file_written` on the verb's mutations record.
- Each leave case of D1 case 4 leaves the entry and the target untouched, succeeds, and reports the shape and the way forward.
- A link creation refused after a successful rename, D1 case 5, leaves the recreated `.scratch` untouched and the moved content at the target, succeeds, and reports the target path and the way forward.
- A failed exclude write creates no link, renames nothing, succeeds and reports the exclude path and the way forward.
- A refused case-1 link creation and a failed target creation, D1 case 6, each succeed, create no link and report the path and the way forward.
- Wiring a worktree whose branch tracks a file under `.scratch/` leaves the directory and its files in place, creates no link and reports the leave case.
- `lyx fabric remove` on a pair with a wired `.scratch` link, at a root anchor and at a subpath anchor, records `link_removed` for the link before the worktree removal and leaves the target's content untouched until the weft teardown.
- `lyx fabric reconcile` converts a junction-healthy pair with a real `.scratch` directory and a legacy `/.scratch/` exclude line, and reports it in that pair's result.
- A second `lyx fabric reconcile` on a converged pair keeps the `.scratch` link and its `/.scratch` exclude line, and reports no stale removal.
- `lyx fabric reconcile` creates no scratch link or target for an unmanaged worktree, or for a pair whose weft worktree is still missing when the scratch step would run; a pair whose weft reconcile recreates in the same pass gets its link.
- `PairComplete` and `fabric status` junction health ignore a missing or real `.scratch`.
- In a fixture whose tracked `.gitignore` holds `**/.scratch/`, the warp exclude holds `/.scratch` and no `/.scratch/` after wiring, and `git status` in the worktree does not list the link.
- `lyx fabric unwire` leaves the `.scratch` link and the `/.scratch` exclude line in place.
- `lyx cleanup`'s scratch segment removes an aged, uncited child of a linked prime `.scratch` and leaves the link itself in place.

## Notes for the plan writer

- `seedLyxJunction`'s create branch is the model for D1's link creation; its verify branch is not, since it resolves the link and fails on a dangling one, and its materialise-first order is not either, since D1 classifies before touching the target.
- `repairPairWiring` is where D3's reconcile call joins, and `applyStaleRemoval` and `Unwire` are where the scratch link's skip joins, never `scanOnDiskJunctionNames`, which `lyx fabric remove` also reads; `Remove`'s junction sweep is where D3's root-link unlink joins.
- Tests that assert the exact entries of a wired worktree's root, or of its warp exclude file, will see the new link and pattern.
- `path_renamed` needs a row in the integration mutation oracle's `manifestObservableKind` (`internal/fabricengine/livestate_mutationoracle_test.go`), which fails loud on an unclassified Kind, and the oracle's omission check must cover the renamed subtree, which is the entry's Detail rather than its Target.
