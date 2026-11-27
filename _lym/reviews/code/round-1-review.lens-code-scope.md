```yaml
lens: code-scope
outcome: reported
```

Task: the code-scope lens, which traces the diff's changes to the goal points of the decision record and the board entry.

**Decisions**
- **G1** (the only goal point, under `## Goal`): when a row is re-stepped after an interrupted-stop halt, compute the prior-work section from the branch's commits beyond the base and from the surviving scratch files, or state that the earlier spawn was interrupted and its work may be present.
- **Added after Discussion entries:** none. The record's other sections are empty.

**Per change**
- **C1:** new `DarnDeps.BranchCommits` and `ScratchDir` seams, `readBranchWork`, `listScratchFiles`, `bulletList`, `priorWorkListLimit`, the call in `darnWrite.Call`, and the no-outcome branch of `renderPriorWork` (`internal/loomshed/darnwrite.go`). Traces to G1. The diff does both halves of the goal's "or": it lists the work and states the interruption.
- **C2:** after a recorded outcome, `renderPriorWork` now adds the commit and scratch lists too (`darnwrite.go`, the `if len(work.commits) &gt; 0` and `if len(work.scratchFiles) &gt; 0` blocks after the `found` branch). Partly traces to G1. G1 asks for the computed section only on a re-step after an interrupted halt; this applies it to every spawn that follows a recorded outcome, such as a resume after a gate-exhausted Stuck. See finding F1.
- **C3:** `darnDeps` wires `BranchCommits` (`taskMergeBase` then `CommitLog`) and `ScratchDir` (`internal/loomcli/darnwiring.go`). Forced by C1.
- **C4:** `fabricengine.ScratchLinkPath` (`internal/fabricengine/scratchlink.go`). Forced by C3.
- **C5:** the `DarnWrite` registry row validates `Darn.ScratchDir` as an absolute path and `Darn.BranchCommits` as non-nil (`internal/shedrecipe/entries_darnwrite.go`). Forced by C1, since the row checks every seam the same way.
- **C6:** the `intentionallyNil` rows in `internal/battencli/wire_seams_test.go` and `internal/loomcli/wiring_seams_test.go`, and the `envkit.FullEnv` fields. Forced by C1, C3 and C5.
- **C7:** the `internal/loomshed/doc.go` lines on prior work. Forced by C1, as the package doc describing changed behaviour.
- **C8:** new rows in `darnwrite_test.go` and `entries_darnwrite_test.go`. Forced by C1, C2 and C5.

**Per decision**
- **G1:** built by C1, with C3 and C4 wiring it, and C2 extending it.

**Findings**
- **F1:** LOW, class `scope`, at `internal/loomshed/darnwrite.go`, the lists `renderPriorWork` appends after the `if found {…}` branch. The commit and scratch lists now follow a recorded outcome, which G1 does not ask for, since it limits the computed section to a re-step after an interrupted-stop halt. The note every resumed spawn gets therefore changes, for example after a gate-exhausted Stuck resume. The extension is plausibly harmless and the summary declares it ("after a recorded outcome, the existing note stands and the same lists follow it"), but G1 does not license it. Evidence: the decision record's `## Goal` text, and the diff hunk at `darnwrite.go` in `renderPriorWork`, the `if len(work.commits) &gt; 0` block.

No goal point is left without a change, and no change lacks a trace or a forcing change apart from F1.

Tool calls: none. My inherited context held the record, the board entry, the diff and the summary.
