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
- Docs: `internal/fabricengine/doc.go` and `internal/cleanupcli/doc.go`.

Out:

- The path agents write to: it stays `.scratch/` at the worktree root, so the edit directive, stencils and skills are unchanged.
- `scankit` and the source-scan tests: they already skip `.scratch` as a walk target, and the root listing they open is a legitimate cache input that must stay hashed.
- Checkouts fabric never wires, such as a plain clone: they keep a real `.scratch`.
- Other root-level churn, such as a `go build` leaving `lyx` in the root or files created directly under `.claude/`.

## Decisions

### D1. `.scratch` is a fabric-wired directory link into the weft's `.lyx`

One fabricengine scratch step owns both the warp worktree's root `.scratch` link and its exclude entry (D4).
Fabric wiring (`WireJunctionsWith`, so worktree add, checkout and the prime's clone-and-wire) runs it after the existing junction and exclude steps, and reconcile runs it per D3.
The link is created through `fslink.CreateDirLink`, and its target is a `scratch` directory inside the weft-side `.lyx` directory the warp `.lyx` link resolves to.
The target path is built from `lyxdirs.DotLyxDirName`; the `scratch` leaf and `.scratch` link name stay fabric's own constants, as `scratchDirName` already is.

The step classifies the root `.scratch` first, and only then touches the target, in this order:

1. Root `.scratch` absent: `MkdirAll` the target, then create the link.
2. Root `.scratch` a link to the target: `MkdirAll` the target, which heals a dangling link after a weft re-creation; nothing else changes.
3. Root `.scratch` a real directory and the target absent: `os.Rename` the directory to the target, then create the link, so its content and every `.scratch/<name>` path survive.
4. Any other shape, the leave cases: a link to another target, a real directory while the target already exists, a regular file, a failed rename, or a link creation refused after a rename because something recreated `.scratch` in between.
   The step leaves the entry untouched and reports the path, the shape and the way forward: move the content into the target and remove the root `.scratch`, then rerun `lyx fabric reconcile`.

Rationale: `os.ReadDir` entries report `Lstat` information, so the hashed entry becomes the link itself, whose modification time is fixed at creation.
Writes through the link change only the target, which lies outside the module root.
The weft `.lyx` is already never tracked and torn down with the pair, so scratch keeps today's lifetime, and agents keep writing `.scratch/<name>` unchanged.
Wiring the link at worktree creation also removes the first `mkdir .scratch` from the root listing's history.

### D2. The scratch step never fails wiring and never removes anything

No outcome of the scratch step fails `WireJunctionsWith`, fails a reconcile pair or refuses a run.
The step removes no file, directory or link: it never re-points a wrong link and never clears a target, so `internal/fabricengine/destroy.go` is not involved.
Every link it creates, every rename and every exclude write is recorded on the verb's `*Mutations` record, per `PATTERN-mutation-record`.
A leave case is reported in the pair's reconcile result detail under reconcile, and logged through `internal/logger` under add, checkout and clone, whose results carry no detail field.

Bound: a leave case keeps that worktree's cache misses and nothing worse, until the operator clears it as the report says.
This covers an agent's `rm -rf .scratch` followed by `mkdir .scratch`: the target still exists, so reconcile reports the real directory each pass and never converges it on its own.

Rationale: scratch is agent convenience and wiring gates worktree creation, so a scratch problem must never block a task or lose content.

### D3. Reconcile converges managed pairs; health checks stay scratch-blind

`checkJunctionHealth`, `PairComplete` and `preflight`'s `Healthy` do not look at `.scratch`.
`Reconcile` runs the scratch step from `repairPairWiring`, on its healthy and its re-wired branch alike, and adds a detail to the pair's result when the step changed or left something.
It never runs the step on a worktree reported as unmanaged, on a raw worktree whose junctions this pass has not wired, or on a pair whose weft worktree root is missing at the time of the step; with no weft root, the step creates nothing.
`lyx fabric unwire` leaves the scratch link and its exclude in place, as it leaves the scratch exclude today; the weft it points into stays, so the link stays valid.

Bound: the step runs only under the wiring verbs of D1 and under `lyx fabric reconcile` on a pair whose weft exists; no status read, preflight or other verb touches `.scratch`.

Rationale: junction health gates runs through `PairComplete` and `preflight`, so a missing or real `.scratch` must not refuse a run.
Reconcile is the existing operator verb for converging old worktrees, so the prime and every pair wired by an older binary convert there: after this lands, the operator runs `lyx fabric reconcile` once, and the fabricengine doc says so.

### D4. The scratch exclude matches the link

The scratch step writes the anchored pattern `/.scratch`, with no trailing slash, and replaces a legacy `/.scratch/` line it finds, the way `seedGitExclude` replaces its legacy bare-name lines.

Rationale: git does not follow a link, and a pattern with a trailing slash matches only a directory, so `/.scratch/` would leave the link untracked dirt in `git status`, in dirty-tree checks and in `git worktree remove`.
`/.scratch` matches the link, a Windows junction and a real directory alike.
Because the scratch step owns the exclude, reconcile rewrites it on healthy pairs too, not only on pairs whose junctions it re-wires.

### D5. Cleanup follows the prime's link

`lyx cleanup`'s scratch segment keeps listing the prime's `.scratch` by that path; `os.ReadDir` follows the link to the target, so the segment judges and removes the target's direct children as before and leaves the link.
Its doc states that the prime's `.scratch` is a fabric link and the segment works on what it points at.

Rationale: the segment's contract is about the children of `.scratch/`, which keep their names under D1.

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

- On Windows, the design assumes a directory junction's own modification time does not move when files are created under its target; the board entry `fabric-windows`, which verifies fabric's junction behaviour on a Windows host, is where this gets measured.

## Acceptance criteria

- Wiring a new pair leaves `.scratch` at the warp root as a link resolving to `scratch` under the weft-side `.lyx` directory.
- After a file is created under the wired `.scratch/`, the `Lstat` of the warp root's `.scratch` entry, modification time included, is unchanged.
- Re-wiring a pair with a correct link changes nothing and records nothing; a correct link whose target is missing gets its target back.
- Wiring a worktree whose `.scratch` is a real directory with files, and whose target is absent, moves the files to the target and leaves `.scratch/<name>` readable through the link.
- Each leave case of D1 leaves the entry and the target untouched, succeeds, and reports the shape and the way forward.
- `lyx fabric reconcile` converts a junction-healthy pair with a real `.scratch` directory and a legacy `/.scratch/` exclude line, and reports it in that pair's result.
- `lyx fabric reconcile` creates nothing for an unmanaged worktree or a pair whose weft worktree is missing.
- `PairComplete` and `fabric status` junction health ignore a missing or real `.scratch`.
- The warp exclude holds `/.scratch` and no `/.scratch/` after wiring, and `git status` in the worktree does not list the link.
- `lyx cleanup`'s scratch segment removes an aged, uncited child of a linked prime `.scratch` and leaves the link itself in place.

## Notes for the plan writer

- `seedLyxJunction`'s create and verify branches are the model for D1's link handling; its materialise-first order is not, since D1 classifies before touching the target.
- `repairPairWiring` is where D3's reconcile call joins.
- Tests that assert the exact entries of a wired worktree's root, or of its warp exclude file, will see the new link and pattern.
