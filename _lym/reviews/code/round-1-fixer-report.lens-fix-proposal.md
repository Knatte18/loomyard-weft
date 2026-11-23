```yaml
lens: fix-proposal
outcome: reported
```

Lens fix-proposal: challenge the reviewer's proposals that the fixer carried out as proposed (F1, F2, F3, F4, F5).

1. **MEDIUM** — `internal/boardengine/store.go`, `MergeTasks` loop `s.removeEntry(slug, slug != slugStr)`, which carries out F2's proposal.
   - **Problem:** a merge that folds an entry into a different slug strips that entry from its dependents instead of keeping the wait.
   - **Case:** the merge removes `a` and `c` and upserts `b`, while task `x` has `depends_on: [a]`. `x` ends with an empty `depends_on` and reads as released, although the merged entry `b` now carries the work.
   - **Evidence:** the F2 fix keeps a dependency only for a slug the merge upserts again, and the new `TestRemoveTaskStripsTheSlugFromDependsOn` merge row covers only the same-slug case.
   - **Compared with before:** the old dangling slug also counted as met in `Dependencies.met`, so the wait was lost before too. It is not a regression in waiting, but the fix now silently discards the dependency where the old validation would later have refused it. A repoint to the upserted slug is the behaviour the merge verb implies.

2. **LOW** — `internal/boardengine/layer.go` (`getDepth`) with the F1 `metBelow` change.
   - **Problem:** a task waiting on a note that has status done is `Pending` and drawn in the README After column, but the note's `layerMap` entry is `doneLayer`, so it adds no depth in `ComputeLayers`.
   - **Evidence:** `getDepth` returns depth 0 for any slug with a non-empty `layerMap`.
   - **Effect:** the dependent can show a letter layer A while a note it waits on is "done". This is an edge case, and a note with status done is itself odd, so it only inconsistently signals the wait.

3. **LOW** — `internal/boardengine/store.go` `Prune`, now that notes can be depended on.
   - **Problem:** a note with status done is pruned as done and stripped from its dependents. So "met only once promoted and lands, or removed" holds only because prune counts as removal, and F3's `doc.go` text does not say so.
   - **Evidence:** the `Prune` code is unchanged by the fix, and `doc.go` states the removal strip but not the done-note prune.
   - **Effect:** minor doc gap.

I found no wrong case in the F1 note-block deletion, F4 or F5 beyond the above. `findCycle` still catches a cycle through a note, and the new `a cycle through a note is refused` row covers it.

Tool calls made:
- `grep ComputeLayers` in `internal/boardengine` returned only a few lines, so I could not tell how layers treat a note dependency.
- `sed -n 30,140p internal/boardengine/layer.go` read the layer code, which my context lacked.
