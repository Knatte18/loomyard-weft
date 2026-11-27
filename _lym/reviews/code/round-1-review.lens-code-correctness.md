```yaml
lens: code-correctness
outcome: reported
```

Lens: code-correctness. I walked each function and branch the diff adds or changes with concrete inputs.

**Per-function list**

- **`darnDeps` → `BranchCommits` closure** (`internal/loomcli/darnwiring.go:200-210`).
  - A `mergeBase()` failure is returned unchanged. `readBranchWork` wraps it with the row name and `Call` returns before the spawn, with no state written. Correct.
  - A `fabricengine.Open` failure is returned. Correct.
  - Base = the fork point with HEAD at the fork point gives `CommitsNotIn` → `rev-list base..HEAD` → an empty slice → no lines. Correct.
  - On a first spawn of a darn run: Record-Write, `commitDescription` and the Drift-Expect seal all commit under `_lym`. `_lym` is a link into the weft worktree, so no code-branch commit exists before the writer, and the list is empty. Correct; the range's `git log` agrees.
  - With a merge record (only reachable after Finalize's merge, when a run is sent back to Darn-Write): the base is `MergeBase(MergedTip)`. The parent's commits are excluded, but the merge commit itself is listed. It is true that the branch holds it, though it is not an earlier spawn's. Acceptable, since the note's list header says only "the task branch holds".
  - Order: `rev-list` default order is reverse chronological, so "newest first" holds for a linear branch. Correct.
  - Standalone run with no origin record: `pairMergeBase` → "the pair has no fabric origin record", so the row halts before the spawn. The darn wiring already sets `VerifyMergeBase = taskMergeBase(location)` unconditionally (`darnwiring.go:249`), and the darn gate form requires it (`internal/shedrecipe/recipe.go:118`). That run therefore already failed at the verify gate, and the new halt only comes earlier. No new breakage.
- **`readBranchWork`** (`darnwrite.go:306-317`).
  - A commit-read error is wrapped "read the task branch's commits", and a listing error is wrapped with the path. Correct.
  - `scratchDir` is copied into the struct for rendering. Correct.
- **`listScratchFiles`** (`darnwrite.go:319-336`).
  - Absent dir, and a dangling `.scratch` link (`os.Stat` gives ErrNotExist): nil, nil. Correct.
  - Link to a directory: `os.DirFS` follows the root link. Correct.
  - A symlink entry inside the tree is not followed and is listed as a file. Correct.
  - Nested tree: slash paths in lexical order. Correct; the test sorts `runs/second.txt` after `measure.md`.
  - The scratch path is a link to a regular file: `WalkDir` calls fn on "." with a non-dir entry, so the list is `["."]` and renders `- .`. Wrong but near-unreachable (NIT, not raised).
  - Unreadable subdirectory, or a subdirectory removed between listing and reading: the `WalkDir` error is passed to fn, fn returns it, and `Call` errors, so the Darn-Write row halts on an advisory listing (finding 3).
  - Large tree: the whole tree is walked and kept in memory before the cap applies.
- **`renderPriorWork`** (`darnwrite.go:347-375`).
  - found=false with nothing listed gives "" → `(none)`. Correct.
  - found=false with commits gives the interrupted note plus the commit list. Correct for the board entry's incident.
  - found=false with scratch files only gives the interrupted note. Wrong when the files come from a row before Darn-Write (finding 1).
  - found=true with an empty `FindingsPath` and no work: the old text is unchanged. Correct.
  - found=true, a recorded Stuck, then a later spawn interrupted: the note says "That spawn ended stuck", plus the findings and the commit list. It never says the latest spawn was interrupted (finding 2).
- **`bulletList`**: 1 item → 1 line; exactly 20 → 20 lines with no count; 22 → 20 lines plus "- and 2 more". Correct.
  - The cut keeps the lexically first entries (finding 4).
  - A file name holding a newline breaks the markdown list (NIT, not raised).
- **`checkSeams`**: a nil `BranchCommits` and an empty `ScratchDir` are each named. Correct.
- **`darnWriteEntry`**: `Darn.ScratchDir` goes through `requireAbsRoot`, and `Darn.BranchCommits` through the nil check. `ScratchLinkPath` joins `AnchorPath()`, which is absolute under `HubPath`. Correct.
- **`fabricengine.ScratchLinkPath(l)`**: equals `scratchLinkPath(l, l.WorktreeName)` for the warp location `darnDeps` gets. Correct.

**Findings**

1. MEDIUM, design, `internal/loomshed/darnwrite.go:347-366` (`renderPriorWork`, the scratch-only branch).
   - Summary: scratch files that a row before Darn-Write leaves make the first writer spawn be told an earlier spawn of its own was interrupted.
   - Cause: the task's `.scratch` link is shared by every agent row of the task. Drift-Expect runs an agent right before Darn-Write (`contracts/recipes/darn-recipe.yaml`, `Drift-Expect` row). Its stencil renders `{{.edit_directive}}` (`contracts/stencils/drift/drift-template-expect.md:17`), which sends scratch files to `.scratch/` (`contracts/stencils/engine/claude/edit-directive.md:16`) and never requires their removal.
   - Effect: with found=false, any surviving file triggers "An earlier spawn of this task was interrupted…" and "continue it instead of starting over". The writer stencil then says the prior work is the writer's own from an earlier spawn and to "never restart the change" (`contracts/stencils/darn/darn-template-write.md`, Step 2). The note's source is not attributable to an earlier writer spawn, so the first spawn gets a false statement, the mirror of the bug being fixed. Files the operator or the orch left there have the same effect.
2. LOW, design, `internal/loomshed/darnwrite.go:354-361` (`renderPriorWork`, the found branch).
   - Case: a recorded outcome (e.g. Stuck) is followed by a spawn interrupted before it recorded one.
   - Summary: the note says "That spawn ended stuck." and points at the kept findings, but never says the most recent spawn was interrupted. The writer reads a stale ending as the last spawn's.
   - Evidence: `latestDarnOutcome` returns the last recorded entry (`internal/loomcli/darnwiring.go:131-166`). An interrupted halt appends no history entry (`internal/shedengine/failedstop.go:85-95` sets only State and Error). The commit list that follows does carry the interrupted spawn's commits, so the work itself is not hidden.
3. LOW, design, `internal/loomshed/darnwrite.go:308-317` and `321-336` (`readBranchWork` / `listScratchFiles`).
   - Summary: an unreadable or vanishing entry anywhere under `.scratch` now halts the Darn-Write row before the spawn. Before the diff, that row never read the directory.
   - Evidence: the `WalkDir` callback returns the passed `err` unchanged (`darnwrite.go:327-329`), and `Call` returns it as a producer error (`darnwrite.go:177-180`). The listing only feeds an advisory note, yet any walk error stops the run.
4. LOW, design, `internal/loomshed/darnwrite.go:319-336` and `378-389` (`listScratchFiles` with `bulletList`).
   - Summary: the cap cuts a lexical, fully recursive listing, so a deep tree crowds out the items a resumed writer needs.
   - Example: one cloned checkout or audit work directory under `.scratch` fills the 20 slots with its own lexically-first files, and the half-finished measurement at the top level falls into "- and N more".
   - Cost: the whole tree is also walked and held in memory with no bound before the cut.

Tool calls:
1. `grep`/`sed` of `internal/loomcli/wiring.go` around `verifyMergeBase`: to see how a standalone run's merge base is wired. The manifest had only the comment, not the definition.
2. `sed`/`grep` of `internal/shedrecipe/recipe.go` and `entries_gate.go`, plus the `VerifyTreeScope` grep in loomcli: to check whether the darn gate already requires a merge base, which settles whether a standalone halt is new.
