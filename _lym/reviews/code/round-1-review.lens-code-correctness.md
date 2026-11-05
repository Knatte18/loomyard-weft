```yaml
lens: code-correctness
outcome: reported
```

Task: review the diff through the code-correctness lens, running each changed function against a concrete input. The first block is the list the lens asks for. The findings follow it.

## Per function or branch: the case I ran

- `seedScratchLinkExclude`, correct:
  - Content `"a\n"` at anchor `backend` gives `"a\n/backend/.scratch\n"`.
  - Empty content gives `"/backend/.scratch\n"`.
  - Content with `/backend/.scratch/` (legacy) gives the legacy line dropped and the pattern appended.
  - Content already holding the pattern gives unchanged content and no write.
  - The trailing `""` element survives the `Split`/`Join` round trip.
- `unseedScratchExclude`, correct for the trailing-newline case:
  - `"a\n/.scratch\n"` becomes `"a\n"`.
  - With no matching line it returns the content untouched and runs no census, so ordinary wiring lists no worktrees.
  - When `scratchLinkedInSibling` is true it returns the content unchanged, so nothing is written and no mutation is recorded. The census runs under the exclude lock.
  - The sibling-census gap is finding 1.
- `scratchLinkedInSibling`:
  - When the prime is the worktree being retired, `ownPath` is the prime, so it is skipped. Its root link is already gone by then. Correct.
  - When `List(WorktreePath(l, slug))` fails, it falls back to `l.WorktreePath()`. If both fail, it answers true, so the line stays. Safe.
  - A pruned entry whose directory is gone gives `IsLink` `(false, nil)`, so it does not pin the line. Correct.
  - A sibling with a real root directory or a regular file returns false. Wrong, see finding 1.
- `retireRootScratchLink`:
  - At anchor `.` it returns an empty outcome. Correct.
  - With the root absent it runs the exclude drop on every subpath wire, but only does a census when the line exists. Correct.
  - With an owned root link, the link is removed, the root is then absent, and the exclude drop runs. Correct.
  - With a real directory or a foreign link at the root, nothing is removed, the exclude line is kept and a way-forward `left` is returned. Correct, so a real directory or another pair's link is never removed.
  - With an `Lstat` error other than not-exist, it reports and keeps the line. Correct.
  - A removal error becomes a `left` string and wiring continues. Correct, since the step never fails wiring.
  - A dangling legacy root link is not retired in the same pass, see finding 3.
- `wireScratchLink`: `retire.and(wireAnchor)` evaluates the retire first, then the anchor wiring runs whatever the retire returned. `and` joins both `changed` and `left` parts. Correct.
- `wireAnchorScratchLink`, `linkAbsentScratch`, `restoreScratchTarget`, `adoptScratchDirectory`: the logic is the old root logic moved to `scratchLinkPath`, with the tracked check using `relLink` built from `AnchorRel`. Correct at anchor `.` and at `backend`.
- `unlinkScratchLink`:
  - At anchor `.` it unlinks one path.
  - At `backend` it unlinks the anchor link and the legacy root link. A non-owned or non-link entry is skipped without error. Correct.
  - A gate refusal returns an error naming the link, which `surfaceRefusal` raises in `Remove`.
  - The order in `removePair` is the junction sweep, then the scratch unlink, then the worktree directory removal. Correct.
- `removeOwnedScratchLink`: `Lstat` not-exist gives `(false, nil)`. A real directory gives `(false, nil)`. A link outside the weft worktree gives `(false, nil)`. An owned link goes through `removeLink`. Correct.
- `withoutScratchLink`:
  - `scanOnDiskJunctionNames` only returns links resolving into the weft worktree, so a user's own `.scratch` file or directory, or a link elsewhere, is never in `names` to begin with. The filter only drops a fabric-owned `.scratch` link.
  - `Unwire` at a root anchor behaves as before.
  - `Unwire` at a subpath anchor now keeps the anchored link, as intended.
- `AnchorChange.unwireAtSource`:
  - From `.` to `backend`: it unlinks the root link and drops the root line unless a sibling still links the root.
  - From `backend` to `.`: it unlinks the anchored link and a legacy root link, then `wireAtTarget` wires at the root.
  - If any unwire step fails, `rollback` calls `WireJunctionsWith` at the old anchor, which re-seeds the exclude line and relinks. The state is consistent.
  - If the failure comes after the weft commit, there is no rollback. The new anchor's wiring is the re-run's job, as for the junctions. No new inconsistency.
  - The new anchor's scratch step runs inside `WireJunctionsWith(target, ...)` in `wireAtTarget`, after the move. Correct.
- `Reconcile` (`repairPairWiring`): the scratch step runs on every managed pair, on both the healthy and the re-wired path, before `applyStaleRemoval`. The scan skips `.scratch`. Correct.
- Cleanup segment (`segments.go:120`): the diff does change code there. It joins `location.AnchorRel` onto `location.WorktreePath()`, so the segment reads `<prime>/<anchor>/.scratch`. The focus note's claim that only comments changed is wrong. The code is correct at anchor `.` and at a subpath anchor.

## Findings

1. **MEDIUM, design.** `scratchLinkedInSibling` counts only a link at the sibling's `<anchorRel>/.scratch`.
   - Location: `internal/fabricengine/scratchlink.go:181-184`.
   - Case run: the prime has a real root `.scratch` directory that was left as found, so its root line is deliberately kept. Pair B, anchored, wires and retires its own root link, the root `/.scratch` line is in the exclude file, the census sees no sibling link, and the line is dropped from the shared file.
   - Result: the prime's root `.scratch` shows as untracked. That contradicts the summary's design note "A root entry fabric does not own keeps its exclude line", which the code holds only for the worktree being wired.
   - Evidence: `scratchlink.go:165-187` and `scratchlink.go:215-218`.

2. **LOW, design.** `retireRootScratchLink` removes the root link before the anchor step can succeed, with no link left at either place if the anchor step then leaves or fails.
   - Location: `scratchlink.go:193-195`, `scratchlink.go:208-214`.
   - Case run: the anchored `.scratch` is tracked, or a regular file, or the exclude write fails. The root link is gone and the root line is dropped, but no anchor link exists.
   - The `left` outcome names the way forward and the target is untouched, so this recovers on a rerun. It is a transient loss of scratch.

3. **LOW, design.** A legacy root link whose weft target is absent is not retired.
   - Location: `scratchlink.go:208-211`, `reconcile.go:811-828`.
   - Case run: `linkIsFabricOwned` uses `PointsTo`, which fails on a dangling link, so it reports not owned. The root is then reported as "not a scratch link of this pair" and its exclude line is kept.
   - The anchor step recreates the same target in that pass, so the link would be retired on the next reconcile. The report text is misleading for this one case.

4. **LOW, design.** `scratchLinkedInSibling` counts any worktree of the repository with a root `.scratch` link, including a user's own worktree and its own link, so such a link keeps the root line permanently.
   - Location: `scratchlink.go:176-185`.
   - The code deliberately trades a dead line against a dirtied link. It is only a note against the goal that an anchored hub has no root line.

## Tool calls

- `grep` on `internal/fabricengine/junction.go` and `internal/fslink/*.go`: I needed `excludePatternFor` and `fslink.IsLink`, which the manifest did not hold.
