# The scratch link lives at the anchor and is excluded there

In an anchored hub, agents work from the anchor and write `.scratch/` there.
Fabric only linked and excluded the warp worktree's root `.scratch`, so the C# sandbox hub `cs` (anchor `dotnet`) showed `?? dotnet/.scratch/` in the prime.

## What changes

- **Wiring.** The scratch step now links `<anchor>/.scratch` to `<weft>/<anchor>/.lym/scratch`, the same target as before, and writes `/<anchor>/.scratch` into the warp `info/exclude` beside the other `/<anchor>/…` lines.
  At the default `.` anchor the path is the root, so hubs without an anchor see no change.
  The adopt, restore and leave-as-found handling is unchanged and now applies at the anchor, so a real `dotnet/.scratch` directory like the one in `cs` is moved into the target and linked.
- **Migration.** At a subpath anchor the step first retires what an earlier binary wired at the root.
  It removes a root `.scratch` link that resolves into the pair's weft worktree through the destructive gate,
  and it drops the root `/.scratch` line once no other warp worktree of the repository still has a root `.scratch` link (a census under the exclude lock, since the file is shared).
  Any other root `.scratch` is left as found, keeps its exclude line and is reported with its way forward.
  `lym fabric reconcile` runs the step on every managed pair, so one pass moves an anchored hub's worktrees and prime over without a hand edit of `.git/info/exclude`.
- **Teardown.** `Remove` unlinks the anchor's link, and at a subpath anchor a legacy root link still standing, ahead of the worktree removal.
  An anchor change also drops the old anchor's `.scratch` exclude line, under the same sibling census.
  The on-disk junction scans in `Reconcile`, `Remove` and `Unwire` now always skip `.scratch`, since the link sits in the anchored directory they scan; `Unwire` keeps the link and its exclude line as before.
- **Cleanup.** `lym cleanup`'s scratch segment reads `.scratch` at the prime's anchor.

## Design decisions

- The root `/.scratch` line is kept while a sibling worktree still has a root link, so wiring one pair never turns another pair's unmigrated link into untracked dirt; the last pair `Reconcile` migrates drops it.
- The census runs only when the exclude file holds a line to drop, so ordinary wiring does not list worktrees.
- A root entry fabric does not own keeps its exclude line: dropping it would surface possibly-agent content as untracked and trip `Remove`'s dirty gate.

## Tests

`TestScratchLink_SubpathAnchor` replaces `TestScratchNameAtSubpathAnchor`, whose premise (a `.scratch` link at a subpath anchor is an ordinary junction) no longer holds.
It pins `Unwire` keeping the anchor link with `/backend/.scratch` and nothing at the root,
and the migration: wiring one pair retires its root link while `/.scratch` stays for the prime's root link, then `Reconcile` retires the prime's and drops the line, leaving `git status` clean.
The `scratchPaths` test helper now returns the anchor's entry, so the existing subpath-anchor wiring and anchored `Remove` rows assert the new location.
