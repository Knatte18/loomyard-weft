# Wire and exclude the scratch link at the anchor, not the root

In an anchored hub, agents work from the anchor and write `.scratch/` there.
Fabric only linked and excluded the warp worktree's root `.scratch`, so the C# sandbox hub `cs` (anchor `dotnet`) showed `?? dotnet/.scratch/` in the prime.

## What changes

- **Wiring.** The scratch step links `<anchor>/.scratch` to `<weft>/<anchor>/.lym/scratch`, the same target as before, and writes `/<anchor>/.scratch` into the warp `info/exclude` beside the other `/<anchor>/…` lines.
  At the default `.` anchor the path is the root, so hubs without an anchor see no change.
  The adopt, restore and leave-as-found handling is unchanged and now applies at the anchor, so a real `dotnet/.scratch` directory like the one in `cs` is moved into the target and linked.
- **Migration.** At a subpath anchor nothing of lym's lands at the repository root, so the step first retires what an earlier binary wired there.
  It removes a root `.scratch` link that resolves into the pair's weft worktree through the destructive gate,
  and it drops the root `/.scratch` line once no other warp worktree of the repository has any entry at its root `.scratch` (a census under the exclude lock, since the file is shared).
  Any other root `.scratch` is left as found, keeps its exclude line and is reported with its way forward.
  `lym fabric reconcile` runs the step on every managed pair, so one pass moves an anchored hub's worktrees and prime over without a hand edit of `.git/info/exclude`.
- **Teardown.** `Remove` unlinks the anchor's link, and at a subpath anchor a legacy root link still standing, ahead of the worktree removal.
  An anchor change also drops the old anchor's `.scratch` exclude line, under the same sibling census.
  The on-disk junction scans in `Reconcile`, `Remove` and `Unwire` now skip `.scratch`, since the link sits in the anchored directory they scan; `Unwire` keeps the link and its exclude line as before.
- **Cleanup.** `lym cleanup`'s scratch segment reads `.scratch` at the prime's anchor.
- **Docs.** `internal/fabricengine/doc.go`, `docs/overview.md`, `internal/cleanupcli/doc.go` and `contracts/specs/loom-plan-spec.md` name the anchor's `.scratch` where they named the root's.

## Design decisions

- The root `/.scratch` line is kept while a sibling worktree still has an entry at its root `.scratch`, so wiring one pair never turns another pair's unmigrated entry into untracked dirt; the last pair `Reconcile` migrates drops it.
- The census runs only when the exclude file holds a line to drop, so ordinary wiring does not list worktrees.
  A worktree list or sibling entry that cannot be read counts as a sibling entry, since a dead line costs less than dirtied content.
- A root entry fabric does not own keeps its exclude line: dropping it would surface possibly-agent content as untracked and trip `Remove`'s dirty gate.

## Tests

`TestScratchLink_SubpathAnchor` replaces `TestScratchNameAtSubpathAnchor`, whose premise (a `.scratch` link at a subpath anchor is an ordinary junction) no longer holds.
It pins `Unwire` keeping the anchor link with `/backend/.scratch` and nothing at the root, and the migration to a clean `git status`.
The integration tests in `internal/fabricengine` and `internal/cleanupcli` also cover the anchor change's exclude drop, the legacy root link under `Remove`, non-fabric root entries and the cleanup segment's anchored scratch path.

## To check by hand

Nothing was run by hand, and no regression check ran.
A review left these findings open; the last fix commits address most of them, so confirm each against the diff:

- `anchor-change-exclude-drop-unpinned`: the anchor change's drop of the old anchor's `.scratch` exclude line in `internal/fabricengine/anchorchange.go` needs a pinning test.
- `census-ignores-sibling-real-root-scratch`: the sibling census in `internal/fabricengine/scratchlink.go` must count a real directory or file at a sibling's root `.scratch`, not only a link.
- `cleanup-anchored-scratch-path-untested`: `lym cleanup`'s scratch segment at a subpath anchor needs a test in `internal/cleanupcli`.
- `doc-go-remove-paragraph-contradicts-code`: the `Remove` paragraph of `internal/fabricengine/doc.go` must match the anchor and legacy root unlink the code does.
- `foreign-root-entry-at-subpath-anchor-untested`: a root `.scratch` fabric does not own, at a subpath anchor, needs a test that it is left as found with its exclude line kept.
- `remove-legacy-root-link-at-subpath-anchor-unpinned`: `Remove` of a legacy root link at a subpath anchor needs a test.
