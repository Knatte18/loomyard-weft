```yaml
lens: fix-consistency
outcome: reported
```

Task: check the change set for agreement with its surroundings (lens fix-consistency).

1. **LOW** — `contracts/stencils/darn/darn-template-write.md` Step 2, and the added bullet in `_lym/landing/summary.md`, "What changes".
   - The bullet says the stencil "no longer says the branch's commits are always the writer's own".
   - The same file's "Design decisions" still says "any code commit beyond the merge base at Darn-Write is the writer's own, and listing it needs no filter".
   - A reviewer would read the two as disagreeing about whether those commits can be someone else's. The stencil change is about scratch entries, which other agents write too, not about the commits.

2. **LOW** — `internal/loomshed/darnwrite.go`, `renderPriorWork`.
   - The list header still reads "These files survive under the scratch directory %s:", and the neutral note says "may have left the files below".
   - The list now holds directory names such as `runs/` and `locked/`, including empty directories. The docs changed in the same commit call these "scratch entries" (`branchWork.scratchEntries`, `priorWorkListLimit`, the `loomshed/doc.go` prior-work lines).
   - The `TestDarnWrite_Call` rows pin the "files" wording over a list holding `- runs/` and `- locked/`.

I found nothing else stale in `loomshed/doc.go`, `darnengine/spec.go`, the `DarnDeps` and `DarnTold` docs or the renamed test rows. A search showed no twin of the prior-work wording elsewhere under `internal`, `contracts`, `docs/overview.md` or `PATTERN*.md`.

Tool calls:
- One `grep` for "scratch file", "scratchFiles", "listScratchFiles" and "prior work" across `internal`, `contracts`, `docs/overview.md` and `PATTERN*.md`. My context's earlier search output was truncated and predated the edits.
