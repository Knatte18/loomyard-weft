```yaml
lens: code-affected-set
outcome: reported
```

Task: the code-affected-set lens. I checked what the diff's changes force outside the lines it touches.

**Per change, the set found**

1. **`parentMergeTip` takes a new `takeOwedLocal bool` parameter.**
   - Callers: `finalize.go:380` passes `true` and `parentalign.go:147` passes `false`. Both are updated.
   - Tests: no test calls `parentMergeTip` directly. `TestMergeIn_MergeTip` (`parentalign_test.go:314`) drives it through `ParentAlign.Call`, so it is covered.
2. **Finalize's lock-free pass merges the local tip of a parent ahead of its upstream by owed landing tips alone.**
   - `parentMergeTip`'s own doc (`parentalign.go:229-230`): not covered, finding 1 below.
   - `landingshed/doc.go:42`: updated, covered.
   - `gateUnlocked` doc (`finalize.go:360-361`): updated, covered.
   - `landUnderLock` doc (`finalize.go:413-414`): still accurate.
   - `Deps.ParentAheadByOwed` doc (`deps.go:180-186`): still accurate but incomplete, finding 2 below.
   - `docs/overview.md:1394-1401` says "merges the parent's tip" in general terms, so it stays true.
   - `pattern/PATTERN-landing.md` has no rule on which tip is merged, so it stays true.
3. **Parent-Align keeps merging the upstream tip.** Its readers are unaffected and correct:
   - `docs/overview.md:1359`
   - `contracts/recipes/tapestry-recipe.yaml:50` and `:332`
   - `doc.go:31`
4. **New error message** `read whether parent branch %q is ahead by owed landings alone`:
   - The new `TestFinalize_LockFreePassStops` row asserts it.
   - It matches the wording of `syncParent` at `finalize.go:725`, so it is consistent.
5. **Commit messages and departures:**
   - The `Root-Cause:` trailer on 704310343 names the defect the diff fixes: `parentMergeTip` returned the upstream tip for every ahead parent.
   - No `Pattern-Departure:` or `Loosened-Check:` trailer is present, and none is needed.
   - No PATTERN changes are declared and the diff makes none.

**Findings**

1. **LOW, consistency, `internal/landingshed/parentalign.go:229-230`.**
   The doc still says that with an upstream it "takes the upstream tip, never the local tip, so a landing another run made locally and has not pushed is never merged in".
   With `takeOwedLocal` set, it now merges exactly such landings, which are owed and not yet pushed. The sentence contradicts lines 232-234 and is false for Finalize.
   Evidence: `parentalign.go:229-234` and the branch at `parentalign.go:247-257`.
2. **NIT, consistency, `internal/landingshed/deps.go:180-183`.**
   The `ParentAheadByOwed` doc gives the seam's one effect as "Such a parent takes the landing as a level one does".
   The seam now also decides which tip Finalize's lock-free pass merges, and the doc does not say so. It is incomplete, not false.
   Evidence: `deps.go:180-186` and its use at `parentalign.go:251`.

**Tool calls**

1. A grep for `parentMergeTip(` callers and for prose naming the upstream tip or lock-free merge across docs and recipes. The context held only the two call sites seen in the diff, not a full search.
2. A grep of those lines in `contracts/recipes/tapestry-recipe.yaml` and `docs/overview.md`, to read the matched recipe prose, which the context lacked.
