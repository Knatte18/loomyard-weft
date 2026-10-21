# fabric: wire .scratch as a link so scratch files keep go test cache hits

Writing files under `.scratch/` in a fabric worktree no longer changes any input `go test` keys its cache on.
Source-scan tests that read the module root keep their cache hits while an agent writes scratch files.

The miss came from the root listing alone.
A test that opens a directory makes the cache key include the `Lstat` of every entry in it, and the `scankit` walk opens the module root, whose real `.scratch` entry changed modification time on every create or remove under it.

Closes #771

## What changes

Fabric wiring now makes a warp worktree's root `.scratch` a directory link, created through `internal/fslink`, into a `scratch` directory inside the pair's weft-side `.lyx` directory.
The root listing then holds a link whose own `Lstat` does not move when files under it change.
Agents still write `.scratch/<name>` at the worktree root, so the edit directive, stencils and skills are unchanged.

- A new scratch step in `internal/fabricengine/scratchlink.go` owns the link and the warp `.git/info/exclude` entry.
  `WireJunctionsWith` runs it on the worktree add, checkout and prime clone-and-wire paths, replacing `seedScratchExclude`.
- `lyx fabric reconcile` converges the link and exclude of every managed pair, healthy or re-wired.
  After this lands, run `lyx fabric reconcile` once so the hub prime and every pair wired by an older binary convert; `internal/fabricengine/doc.go` says so.
- `lyx fabric remove` unlinks the root `.scratch` link, recorded as `link_removed`, before it removes the warp worktree directory.
  This works at a root and at a subpath anchor.
- `lyx cleanup`'s scratch segment keeps working on the prime's `.scratch`: it follows the link and judges the target's children.
  `internal/cleanupcli/doc.go` states this.

## Design decisions

- **Classify, then touch the target.**
  The step writes the exclude first, then classifies the root `.scratch`.
  Absent: create the target and the link.
  A link whose literal target, read through `fslink.RawTarget`, matches: only `MkdirAll` the target, which heals a dangling link after a weft re-creation.
  A real directory with no tracked path and an absent target: `os.Rename` it to the target, then link, so content and `.scratch/<name>` paths survive.
  Tracked status comes from `PathTracked`, which runs `git ls-files -- .scratch`.
- **The step never fails wiring and never removes anything.**
  A scratch problem must not block a task or delete content.
  Every other shape is left untouched and reported with the shape and the way forward: a link to another target, a real directory while the target exists, a regular file, a failed rename, a tracked `.scratch`, a link refused after a rename, a failed target creation, a failed exclude write.
  Reconcile reports it in the pair's result detail; add, checkout and clone log it through `internal/logger`.
- **The exclude is `/.scratch` with no trailing slash.**
  A trailing-slash pattern matches only a directory, so `/.scratch/` would leave the link as untracked dirt.
  The tracked `.gitignore` line `**/.scratch/` also makes a `git check-ignore .scratch/` probe answer ignored for a link it does not cover, so the step writes unconditionally and replaces a legacy `/.scratch/` line.
- **Health checks stay scratch-blind.**
  `checkJunctionHealth`, `PairComplete` and `preflight` ignore `.scratch`, so a missing or real `.scratch` never refuses a run.
  The stale-junction sweep, `applyStaleRemoval` and `lyx fabric unwire` never claim the link or strip its exclude line.
- **New mutation kind `path_renamed`.**
  Its Target is the target path and its Detail is the renamed root `.scratch`.
  The links record as `link_created` and exclude writes as `file_written`.
  The integration mutation oracle classifies the new kind and checks the renamed subtree.
- A tracked or tracked-content `.scratch`, and a `.scratch` in a checkout fabric never wires, such as a plain clone, stay real directories.
- Other root-level churn, such as a `go build` leaving `lyx` in the root, is out of scope.

## To check by hand

- Windows: the design assumes a directory junction's own modification time does not move when files are created under its target.
  Nothing here ran on Windows; the check is appended to the `fabric-windows` board entry.
- Run `lyx fabric reconcile` on an existing hub and confirm the prime's `.scratch` converts to a link and `git status` stays clean.
- Run `lyx gate test --tags integration ./...` in a wired worktree, create a file under `.scratch/`, and confirm the source-scan packages hit the cache on a second run.
- An agent's `rm -rf .scratch` followed by `mkdir .scratch` leaves a real directory next to the existing target.
  Reconcile reports it each pass and does not converge it on its own.
