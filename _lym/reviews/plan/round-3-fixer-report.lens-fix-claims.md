```yaml
lens: fix-claims
outcome: reported
```

Task: check every claim the fix added or rewrote against the code at HEAD (fix-claims lens).

**Findings (3)**

1. **MEDIUM.** Card 12 (`12-code-tip-grace.md`), the rewritten `ReturnToPreviousTransition` paragraph: "commits it through `CommitAnchoredPathsWithTrailers` with the same `Run-Transition` trailer, so its `Warp-SHA` names the code tip that transition names."
   - **Problem:** the "so" claim is false. `CommitAnchoredPathsWithTrailers` takes `Warp-SHA` from the local warp checkout's HEAD, not from the previous transition.
   - **Evidence:** in `internal/fabricengine/trailercommit.go:99-108`, `messageWithTrailers` calls `f.HeadSHA()` and then `appendWarpSHATrailer(message, warpSHA)`. `HeadSHA` returns `f.code.CurrentSHA()` (`warpforward.go:26`).
   - **Consequence:** the owner's pair HEAD is whatever its local code branch holds, so it matches the previous transition's tip only by coincidence. If the give-up commit names a different tip, the next read can find a new missing or wrong tip.
   - **What it needs:** say how that tip is made the Warp-SHA, for example by moving the local code branch to it first or by writing the trailer explicitly. Or drop the "so" claim.

2. **LOW.** Overview, the `code-tip-is-warp-sha` rationale: "Fabric writes `Warp-SHA` into every weft commit (`commitWeftLocked`, so `lym fabric commit`, `lym fabric sync` and the anchor re-anchor commit too)".
   - **Problem:** the re-anchor commit does not go through `commitWeftLocked`.
   - **Evidence:** `pull.go:773-774` builds the message with `appendWarpSHATrailer` itself and commits through `f.commitEmptySnapshot`.
   - **Problem:** "every weft commit" is also too broad. Card 8's own unchanged text says "`CommitAnchoredPaths` writes no trailer", and `CommitAnchoredPathsWithTrailers` (`trailercommit.go:77-93`) adds the trailers before it calls `CommitAnchoredPaths`.

3. **LOW.** Card 6's new paragraph: "skipping fabric's other weft commits, which carry `Warp-SHA` alone."
   - **Problem:** same overstatement. Some weft commits carry no trailer at all, namely those written through `CommitAnchoredPaths` (card 8: "writes no trailer").
   - **Impact:** the selection rule is unaffected, because the selector keys on `Run-Transition`. Only the stated fact is wrong.

**Checked and holding:**
- `SetArchiveTagPushHookForTest` restores nil in `t.Cleanup` (`export_test.go:637-644`).
- `shedrun.ResolveRunID` exists (`resolve.go:20`).
- `lym fabric commit` and `lym fabric sync` go through `fab.Commit` (`weft_verbs.go:198,280`). That backs card 7's new row with a "`lym fabric commit` style commit carrying `Warp-SHA`".
- `WarpSHATrailerKey` is in `trailer.go`, so placing `TransitionTrailerKey` beside it is accurate.

**Tool calls:**
- One Bash call read `pull.go:765-780`, `weft_verbs.go:192-202` and `275-284`, `recordsgit.go:200-215`, and the `Fabric.HeadSHA` body. My context lacked the re-anchor commit's path, the fabric verbs' commit call and what `HeadSHA` returns.
