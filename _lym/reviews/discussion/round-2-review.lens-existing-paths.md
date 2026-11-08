```yaml
lens: existing-paths
outcome: reported
```

Lens: existing-paths. I traced each decision's state through the code that already writes or reads it.

**Paths traced, per decision**

- **D1**
  - The `undiscussed` gate skips Discussion on restart. Survives.
  - `finishesAsSketch` sees the RaddleFile row and sends no Sketch write on Done (`internal/loomcli/wiring.go:290-292`). Survives.
- **D2 / D4: perch re-arm and its budget**
  - Shed's segment count, `Routing.Bounces` → `episodeStuckCount`, ends the episode at the Bouncer's own Done (`internal/shedengine/run.go:666`). After a return, Shed's count is fresh. Survives.
  - The Bouncer's own re-entry cases, settle record and generation record (`internal/shedadapters/bouncer.go:348-356`). **Breaks**, see finding 1.
- **D4**
  - `StoreCold` stores the cold proposal in any mode but interactive (`internal/loomcli/raddlewiring.go:122-131`). Perch mode stores. Survives.
  - `clearLastAttempt` has only an auto branch and an else branch (`internal/loomshed/raddlewrite.go:134-145`). The record says perch mode moves the proposal aside "as in auto mode". Survives as stated.
  - `verify` matches the approval only in interactive mode (`internal/loomshed/raddlefile.go:200`). Survives.
  - `freshRaddleEntry` tells a return or a goto from its last history entry (`internal/loomcli/raddlewiring.go:57-67`). Survives.
- **D5 / D6: the filing write**
  - `checkProjectField` refuses joining a project a run holds, judged on the after-state (`internal/boardengine/project.go:99`). The converted entry has no status after the write, so members may join. Survives.
  - `fileRaddleEntry` drops any dependency on RunEntry (`internal/boardengine/raddlefile.go:149`). Survives.
  - On retry, `raddleFilingHeld` / `isRaddleFiledFrom` match the body prefix (`internal/boardengine/raddlefile.go:222-240`). The converted body opens with `RaddleProjectBodyPrefix`. Survives.
  - `FileRaddle` removes record files on a failed write (`internal/boardengine/raddlefile.go:125-129`). D6 restores the prior record. Survives as stated.
  - `StoreDone` and `writeRaddleStore` are keyed on the Sketch, which is the arras slug. Survives.
- **D6: the status skip mark**
  - Loom's `SetBoardStatus` → `SetRunStatus` on every transition, and nothing on Done (`internal/loomcli/wiring.go:220-226`, `:310-327`). **Breaks** on resume between the board write and the mark write, see finding 2.
  - Batten's `SetBoardStatus` writes only for rows before Run-Shed (`internal/battencli/commitstatus.go:181`). Survives.
  - Batten cancel's `ClearClaim` writes back the status it read; the converted project's is nil (`internal/battencli/cancelcmd.go:302-320`). Survives.
  - `Cleanup` and `ClearMotifRun` remove the run directory, mark included; the run is done by then. Survives.
  - `RunDir` is keyed on the worktree slug from the anchor, so the loom hook and `ClearMotifRun` read the same directory (`internal/shedrun/paths.go:143`). Survives.
- **D7**
  - `decision add` reads the current row and recipe (`internal/loomcli/decision.go:729-756`). Survives.
- **D8**
  - The other null-status callers of `Board.SetStatus`: fronttask `Release`, beamcli `deps.go:238`, the cancel claim clear, the pairteardown claim. D8 keeps the release out of `Board.SetStatus`. Survives.
- **D10**
  - `Dependencies.metBelow` needs `len(members) &gt; 0` (`internal/boardengine/dependency.go:60`). Survives.

**Findings**

1. **MEDIUM — design — D2, "A return from Raddle-File re-arms the perch".**
   Clearing the perch's settle record alone does not give a fresh cycle with the segment's full budget.
   The Bouncer's re-entry reads two records (`internal/shedadapters/bouncer.go:348-356`):
   - With a Subject seam, a generation record survives the clear. The new proposal differs from the judge subject, so case 3 starts a new generation, as case 2 does (`internal/shedadapters/amendment.go:30`). A new generation runs under the amendment allowance: `budgetReached` caps the budget at `AmendAllowance` (`internal/shedadapters/bouncer.go:1486-1488`), and `atHardCap` caps rounds at `AmendAllowance+1` (`internal/shedadapters/bouncer.go:1294-1302`).
   - Without a Subject seam, the Bouncer keeps no settle record (`internal/shedadapters/bouncer.go:348`). Case 4's clear then fires anyway, and the stated mechanism is empty.

   The record names neither the generation record nor whether the perch has a Subject seam.
   It should state both records the re-arm clears, or rely on case 4's clear with no Subject seam.
2. **MEDIUM — design — D6, the status skip keyed on a filing mark "which Raddle-File writes when its board write lands".**
   A crash or a failed mark write between the board write and the mark leaves a run status stuck on the converted project.
   1. On resume, loom's transition hook runs before Raddle-File's Call. Its first transition in the new process calls `SetRunStatus("running · Raddle-File")` on the converted project, which has no mark yet (`internal/loomcli/wiring.go:220-226`, `:320-326`). `SetRunStatus` writes onto a non-done entry, and no run lock applies to a nil-status before-state (`internal/boardengine/board.go` `SetRunStatus`, `internal/boardengine/runlock.go:159`).
   2. The retry then writes only the store and the mark.
   3. The Done transition writes no board status (`internal/loomcli/wiring.go:312-318`), so the run status stays on the project.
   4. While it stays, `RunHolds` holds the project: `checkProjectField` refuses members (`internal/boardengine/project.go:99`), dobby's `RunHolds` read holds members (`internal/hubgeom/dobbygeom.go:145`), and D8 refuses the release of a project with a run status.

   The record is silent on this ordering.
   The mark should be written before the board write and cleared on a send-back, or the resume path should clear the status.

**Tool calls**

1. grep of `RunHolds` callers and the Bouncer's budget and hard-cap functions: the context lacked the membership validator and the Bouncer's budget rules.
2. `internal/boardengine/project.go` `checkProjectField`, the settle-record grep and the Bounces wiring: the context lacked the held-project member check and the settle record's definition.
3. `internal/shedadapters/bouncer.go:345-372` and `segmentBounces`: the context lacked the Bouncer's re-entry cases and the source of Shed's count.
4. `Routing.Bounces` in `internal/shedengine/progress.go`: needed to see how the segment count is scoped.
5. `episodeStuckCount` in `internal/shedengine/run.go`: needed to see where an episode ends.
6. `internal/shedrun/paths.go` `RunDir`: needed to check that the loom hook and `ClearMotifRun` share the mark's directory.
