Task: fix-code-correctness lens over the F1, F2 and F3 code changes, with the F4 docs excluded as not code.

**F1, `surfaceBeamUpWarnings(notify, slug, warnings)` and `TestSurfaceBeamUpWarnings`**
- Run step by step, the three rows behave as intended.
  - With nil warnings, `beamUpWarningLine` returns "" and the function returns before calling `notify`, so no post.
  - With two warnings, one post carries the joined text.
  - With a `notify` that errors, the function logs and returns, and the post is still recorded.
- Caller: `pawlDeps`'s Beam-Up step is the only caller I saw, and it was updated to pass `notify`. `Deps.Notify` now reuses the same closure, so its behaviour is unchanged. `lymcwd` and `location` are still used in `pawlDeps`, so there is no unused import or variable.
- The test fails if the function stops posting or posts the wrong text.
- LOW: the "AFailedPostReturnsWithoutHaltingTheRow" row cannot fail on its stated property. `surfaceBeamUpWarnings` has no return value, so propagating the error would be a compile error, not a test failure. The row only pins that a post is attempted. The test also does not pin that the step closure calls `surfaceBeamUpWarnings` at all, since reverting the step to ignore `report.Warnings` still passes. This is the seam the review asked for, so I'm reporting it, not asking for a change.

**F2, two rows added to `TestStart_EnsuresTheRunsStrandBeforeTheOrch`**
- Input: `[loom-status s1 live, runs r1 dead]`.
  - The loop's first `case` is false for s1, the second matches it as `loomStatus`, and r1 becomes `runs`.
  - The switch's second case is true (r1 is dead), so `ReplaceStrand(r1)` runs, `replaced.replaced == "r1"` holds and the sidecar reads `replaced-runs`.
- Reversed order gives the same result.
- If `loomStatus` took precedence, it would replace s1, and the loop's `replaced.replaced != "r1"` check would fail. The rows catch the defect.

**F3, new smoke subtest "start beside a runs strand adds the status strand in its own window"**
- With a runs strand in the overview slot, `ensureStatusStrand` finds no status strand and adds one with `statusStrandAddSpec(cmd, statusResult.Strands)`, which gives `AnchorOwnWindow`. The window ids then differ.
- With the strands argument dropped (nil), the spec would take `AnchorOverview` again. The review says reed refuses a second overview strand, so `AddStrand` would fail and `start` would error. The status count would be 0 and `t.Fatalf` would fire, so the test fails with the defect in place.
- The `strandPane` refactor keeps `statusStrandPane`'s behaviour.
- The imports `orchcli` and `render` are used. I saw no cycle: `bootstrap.go` in the same package already imports `orchcli`.
- Caveat: the Tier 2 run reported 240 skipped, so I cannot confirm from the context that this subtest actually ran.

Tool calls made: none; my context already held every changed file.
