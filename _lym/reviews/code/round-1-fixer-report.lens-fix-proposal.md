```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the proposals the fixer carried out as proposed. Those are F5, F7 and F9; the others were changed from the proposal.

1. **LOW (F7): an empty subdirectory now counts as surviving work.**
   - **Changed text:** `listScratchEntries` and the `renderPriorWork` default branch in `internal/loomshed/darnwrite.go`.
   - **Case:** no recorded outcome and no commits, but `.scratch` holds an empty `runs/`, for example left by an agent that removed its files. The old walk listed no files, so the prior work was `(none)`. Now the first spawn is told "may have left the files below", then "These files survive under the scratch directory …:" with `- runs/`.
   - **Effect:** no file survives, yet the stencil's `(none)` sentence no longer applies. The header and the stencil's "commits and files it lists" also call directory entries files.
   - **Evidence:** `if entry.IsDir() { name += "/" }` appends every directory without looking inside it, and the header is still `"These files survive under the scratch directory %s:"`.

2. **LOW (F5): the "later spawn may have been interrupted" line appears on ordinary resumes too.**
   - **Changed text:** the `case found:` branch of `renderPriorWork`, together with the new second stencil sentence in `contracts/stencils/darn/darn-template-write.md`.
   - **Case:** a spawn commits, ends Stuck on its gate, and the run is resumed with no interruption at all. `len(work.commits) > 0` holds, so the line is added. The stencil then tells the writer to treat it as "interrupted or may have been … since that spawn recorded no outcome".
   - **Effect:** any spawn that committed triggers the line, so it cannot tell an interrupted resume from a normal one. It is hedged, so not false, but the stencil's "that spawn recorded no outcome" states as fact something that may refer to no spawn at all.
   - **Evidence:** the condition is `if len(work.commits) > 0 || len(work.scratchEntries) > 0` and nothing else.

F9: no findings.

Tool calls made: none.
