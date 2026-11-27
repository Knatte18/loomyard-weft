```yaml
lens: code-affected-set
outcome: reported
```

Task: the code-affected-set lens. For each change the diff makes, find every caller, test, doc, stencil and rule that reads it, and report any the diff leaves wrong.

**Per change: the affected set found, and whether the diff covers it**

1. **`DarnDeps.BranchCommits` and `DarnDeps.ScratchDir`**, the new seams and their checks in `checkSeams`.
   - The only production builder, `loomcli.darnDeps` (`internal/loomcli/darnwiring.go:185`, called at line 314), is updated. Covered.
   - Test builders: the fixture in `internal/loomshed/darnwrite_test.go`, `darnTestEnv` in `internal/shedrecipe/entries_darnwrite_test.go`, and `FullEnv` in `internal/testkit/envkit/envkit.go:584` are all updated. Covered.
   - `internal/loomrecipe/darnrecipe_test.go:31` builds an empty `DarnDeps{}` only to take its type. Not affected.
   - The registry row in `internal/shedrecipe/entries_darnwrite.go` refuses both new seams when unwired. Covered.
   - The `intentionallyNil` maps in both seam-wiring tests (loomcli and battencli) gain `Darn.BranchCommits`. Covered.
2. **`renderPriorWork`'s new output** (the interrupted note, the commit list and the scratch-file list).
   - `darnengine.DarnInputs.PriorWork` and `orNone` (`internal/darnengine/spec.go:64,156`) receive it through `told.PriorWork` (`darnwiring.go:342`). The field's doc comment is left stale: finding F1.
   - The writer stencil `contracts/stencils/darn/darn-template-write.md`, Step 2, interprets any prior work other than `(none)`. It is left inconsistent with the new note: finding F2.
   - The package doc `internal/loomshed/doc.go:99-100` is updated. Covered.
   - The tests `TestDarnWrite_Call` and `TestRenderPriorWork` are updated, including the literal that replaced a `renderPriorWork` call. Covered.
   - `docs/research/.../data/prompt.md:127-128` quotes the stencil, but it is a measurement record and exempt.
3. **`fabricengine.ScratchLinkPath`** (new).
   - Its one caller is `darnDeps`. Covered.
   - The existing unexported `scratchLinkPath(l, slug)` in `scratchlink.go:63` computes the same path for the worktree named by `slug`. The two now sit side by side: NIT N1.
   - Rules: PATTERN-lym-files allows it, because it names `.scratch` and not `_lym` or `.lym`. It reuses the named constant `scratchDirName` (`gitexclude.go:129`), so PATTERN-code-conventions holds. PATTERN-fabric-boundary holds, because the git reads in loomcli go through `fabricengine.Open` and `CommitLog`.
4. **The test helper `writeLinkedScratch` uses `os.Symlink`.** PATTERN-lym-files exempts test files from the fslink-only rule (`pattern/PATTERN-lym-files.md:120-121`). Covered.
5. **Commit messages.**
   - `Root-Cause:` on `21c033d04` names the defect: the prior work was decided from the history alone. Not a symptom. Holds.
   - `Loosened-Check:` on `0ac64c01b`: the change adds a key for a seam that did not exist at the review base, so no existing assertion was loosened. The trailers are harmless.
   - No `Pattern-Departure:` trailer. No PATTERN changes are declared, and the diff touches neither `PATTERN.md` nor `pattern/`. Holds.
6. **`## Merge danger` in the change description**, which says the blast radius is `loomshed`. The diff also changes behaviour in loomcli, fabricengine and the shedrecipe registry row: finding F3.

**Findings**

- **F1 (LOW, consistency).** Location: `internal/darnengine/spec.go:64`.
  - The `PriorWork` field doc still reads "names what an earlier spawn left unfinished or broken; empty on a first spawn".
  - After the diff, a first spawn gets a non-empty note whenever the scratch directory or the branch beyond the base holds anything (`renderPriorWork`, `internal/loomshed/darnwrite.go:347-349`).
  - The note can also be a "may be present" statement rather than a named outcome, so the field doc is no longer true of what it carries.
- **F2 (MEDIUM, consistency).** Location: `contracts/stencils/darn/darn-template-write.md`, Step 2. The stencil itself is unchanged.
  - It says "When the prior work is not `(none)`, the commits already on the task branch are yours from an earlier spawn".
  - The new note only says "its work may be present on the task branch and in the scratch directory" (`darnwrite.go`, the `else` branch of `renderPriorWork`).
  - The stencil states as certain what the note offers as a likelihood. It mentions no scratch files.
  - In the scratch-only case, it still speaks of the commits as the writer's own when the note lists none.
  - The board entry asks that the prompt "state that the earlier spawn was interrupted and its work may be present". The prompt as a whole now asserts ownership more strongly than that.
- **F3 (MEDIUM, consistency).** Location: `_lym/landing/summary.md`, `## Merge danger`.
  - It gives the blast radius as `loomshed` alone.
  - The diff changes `loomcli.darnDeps` (`internal/loomcli/darnwiring.go:186,200-211`). Each Darn-Write call now reads the merge base through `taskMergeBase`, and on the fallback path `pairMergeBase` and `readRecordedParentBranch` (`internal/loomcli/start.go:186-195`) error when there is no origin record. That failure now halts the row.
  - It adds the exported `fabricengine.ScratchLinkPath`.
  - It tightens the shedrecipe `DarnWrite` registry row, which now refuses construction without the two new seams (`internal/shedrecipe/entries_darnwrite.go:57,69`).
  - So the radius names less than the diff can break.
- **N1 (NIT, consistency).** Location: `internal/fabricengine/scratchlink.go:67-70`.
  - The new `ScratchLinkPath(l)` and the existing `scratchLinkPath(l, slug)` (line 63) build the same anchor-relative path in two places. `scratchLinkPath` is taken through `WorktreePath(l, slug)` and `ScratchLinkPath` through `l.AnchorPath()`.
  - Neither is defined in terms of the other.

**Tool calls**

1. A grep of docs, contracts, `internal/darnengine` and the pattern files for prior-work wording, plus a search for every `DarnDeps{` builder. The manifest had neither the darnengine field doc nor the full list of builders.
2. The `DarnDeps` use in `internal/loomrecipe/darnrecipe_test.go` and the prior-work lines of the darnengine and loomshed docs. That test builder was new to the context.
3. The fslink rule and the named-constants rule in `pattern/PATTERN-lym-files.md` and `pattern/PATTERN-code-conventions.md`, to judge the test's `os.Symlink` and `ScratchLinkPath`. The manifest held only the head of PATTERN-lym-files.
