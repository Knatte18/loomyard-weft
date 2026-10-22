---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: decision
    location: internal/landingshed/finalize.go (parkedMergeWayForward); _lyx/discussion/decision-record.md D2, merge-in-progress way forward
    summary: The merge-in-progress way forward tells the operator to save a copy of their change outside the checkout before --abort, where D2 decides they commit or set it aside with plain git, and the decision record carries no Added after Discussion entry adopting the change.
    fix: The run's parent or the operator decides whether D2 adopts the shipped wording; if so, add an Added after Discussion entry under D2 in _lyx/discussion/decision-record.md recording that the pre-abort step reads "save a copy of any change of their own in <checkout> outside it", because a plain git commit there moves HEAD off the merge record's start so sideConcludeMayHaveLanded makes --abort refuse with merge conclude already landed, and a git stash takes the staged merge with it; if not, parkedMergeWayForward in internal/landingshed/finalize.go returns to D2's wording.
---

### [BLOCKING] The merge-in-progress way forward departs from D2 with no recorded decision

**Location:** `internal/landingshed/finalize.go`, `parkedMergeWayForward`; `_lyx/discussion/decision-record.md`, D2.
**Issue:** D2 fixes the merge-in-progress way forward: before `--abort`, "the operator first commits or sets aside with plain git any change of their own in `<checkout>`".
The shipped text says the operator "first saves a copy of any change of their own in <checkout> outside it, since a plain git commit or stash there would take the staged merge with it".
The plan's overview marks this as a departure from D2 "pending an `Added after Discussion` entry under D2 in the decision record".
Plan-Review round 1 raised it as F1 and carried it over open, and its fixer deferred the entry to the loop owner or `ly:orch`, since the decision record is outside the fixer's write surface.
The decision record at this commit still holds D2's original wording and no such entry, so the change breaks a recorded decision.
The departure looks technically right: Plan-Review confirmed that D2's commit step moves HEAD off the record's start, after which `sideConcludeMayHaveLanded` (`internal/fabricengine/mergeguards.go`) makes `--abort` refuse.
Every element of the acceptance criterion for this form is still carried: the `--abort` reset warning, the role-guard clause, the goto and the resume.
Adopting a change to a recorded decision is the parent's call, not this gate's, so this escalates rather than rejects: a reject would send the fixer back to a way forward that defeats itself.

## Alignment with the goal and the decisions

Apart from F1, the diff matches the board entry's three points and D1–D5:

- D1: no exclusion is added; `internal/landingshed/doc.go` records why run records never count as dirt.
- D2: `MergeGuardError` gains `Checkout` and `DirtyPaths`, which `withDirt` fills at `Merge`, `MergeIn` and the under-lock re-check, and `Error()` stays fixed.
  `pairDirtyReason` decides on the path list.
  Finalize renders four forms: merge in progress (error text, checkout, parked verb and source from `Fabric.MidMerge`, and the no-parked-merge and read-failure branches), detached HEAD (checkout step first), not synced (dirt step, then sync), and dirt only.
  The cap is 20 with Go-quoted paths, and the refusal spec's item 3 is widened in the same commit.
- D3: the halt note's `parent fabric status:` section, the `parentDirtyPaths` seam, `parentDirtyPathsFromFabric`, `renderParentFabricStatus` and `IsWorktreeDirtyReason` are gone, and `Parse` still skips the old header.
  The friction reflection stencil drops its sentence about that section, which follows from D3.
- D4: `finalizeDirtGuard` identifies Finalize by engine (`loomrecipe.IsFinalizeRow`), reads only the code checkout through `Fabric.CodeDirtyPaths`, admits on a read error or a merge record, and is composed ahead of the settled-segment arm.
- D5: `sheddriver.Deps.StartCheck` runs at a blocked start with no open attempt, and `loomcli` wires it to the Finalize arm alone.

The Discussion-Review carry-overs `checkout-step-ordered-after-dirt-step` and `merge-in-progress-arm-names-no-checkout` are settled in the record's D2, and the code follows the settled text.
The recorded open risks (one tackler spawn per dirty halt, an orch goto onto a row routing into Finalize, dirt appearing after the check) are accepted residuals, and the diff does not widen them.

## Overlap

- Peer `run-process-ownership` (row 7; this run's `own_row` is 21): `lyx loom peers` lists no changed paths for it.
  Its decision record concerns process owner records, teardown, pauses and Finalize's parent lock past the parent-side merge.
  It does not touch the merge dirty guard, the goto guard, the driver's start or halt notes, so there is no design conflict.
  By the landing-order rule this run, further along, lands first, and `run-process-ownership` takes its changes at Merge-In.
- Parent commits: none listed.
- Board entries `loop-exit-halt-names-its-cause` (halt-note reason for a loop exit) and `finalize-discrepancy-doc` (conflict-document shape) touch neighbouring surfaces but neither the dirty-parent behaviour nor the `parent fabric status:` section, so neither overlaps.
