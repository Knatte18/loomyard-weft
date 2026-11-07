```yaml
lens: code-correctness
outcome: reported
```

Lens: code-correctness. I ran each changed function or branch on a concrete input.

**Case list**

- `withMissingRecordsUndiscussed`, nil or empty `tasks`: `slices.Clone(nil)` is nil, the loop does nothing, and `Render(nil)` behaves as before. Correct.
- `withMissingRecordsUndiscussed`, unflagged entry: skipped with no `Stat`. Correct.
- `withMissingRecordsUndiscussed`, flagged entry with the file present: the flag stays. Correct.
- `withMissingRecordsUndiscussed`, flagged entry with the file missing: the flag is cleared in the clone only, and the shallow clone shares the `Labels`, `DependsOn` and `RunSettings` slices and maps with the original, which nothing mutates. The caller's `store.Tasks()` keeps `Discussed=true`, so `board.json` is unchanged. Correct.
- `withMissingRecordsUndiscussed`, `Stat` fails with something other than not-exist (permission, I/O): the flag is cleared and the link dropped even though the file may exist. This follows `Board.recordStored` in `decision.go:68`, which also reads any error as "not stored", so it is consistent. It is a transient render difference only, because the next successful render restores the link. Not a defect; at most a NIT (see finding 1).
- `RenderToDisk` with the filtered clone: `Render` receives the clone, so `RenderOrder`, `newBoardView` (README rows) and `renderDesigns` (`render.go:126`) all see the cleared flag. Design doc and row agree. Correct.
- `renderOwnerWrite` (`ownerset.go:246`): it calls `Render` on the unfiltered tasks, then `RenderToDisk`. The unfiltered result is used only for file names, and the flag never changes which files exist or what they are called, so the returned paths are correct. The first `Render` is redundant but harmless.
- `renderDesigns` with a discussed entry that has a body and a dependency: the header joins to `# T\n\nmeta\n\nDecision record: [Sketch](decision-s.md)\n\nDepends on: …\n`, then `"\n"` and the body. That gives the blank-line separation the golden expects. Correct.
- `renderDesigns` with a discussed entry and an empty body: skipped, so there is no design doc and the README row is the only link. Matches the design decision. Correct.
- `renderDesigns` with a discussed project or raddle part: the line is added whenever there is a body. Correct.
- `sketchLink` and `slugColumnCell`: the output is identical to the old inline string. Correct.
- Order of record write and render, board write: `boardCriticalSection` runs `followDecisionRecords` (rename or remove) before `RenderToDisk` (`board.go:195-197`). A renamed entry whose file moved is linked. A rename whose move failed leaves the file at the old name, and the new slug is now unlinked instead of dangling. Correct.
- Order of record write and render, `StoreDecisionRecord`: it writes the file inside the critical section before the render. Correct.
- Order of record write and render, replay: decision files are applied at `replay.go:437`, before the `RenderToDisk` at `replay.go:471`. A decision record "lost" because origin lacks the entry leaves no file, and a flagged entry from origin without the file renders unlinked, which is the intended rule. Correct.
- Manifest: the decision file is not a render output, so cleanup never deletes it. Unchanged. Correct.

**Findings**

1. NIT, `design`, `internal/boardengine/render.go:70`. Any `Stat` error, not only not-exist, drops the link. This matches the existing `Board.recordStored` reading (`decision.go:68`) and heals on the next render, so I recommend no change. Evidence: `render.go:70-72`, `decision.go:66-70`.

I found no other correctness defect in my lens.

**Tool calls**

1. `grep` on `replay.go`: I needed to confirm that replay writes decision files before its `RenderToDisk`, which the manifest did not show.
