---
verdict: APPROVED
findings:
  - id: F1
    severity: NIT
    class: consistency
    location: internal/landingshed/fixerrun.go:298
    summary: haltReason doc opens by saying each reason opens with LandingStopPrefix, which a loom or darn escalate no longer does
    fix: In the `haltReason` doc comment, change the first sentence to "Every reason except a loom or darn escalate opens with LandingStopPrefix and ends with a way forward, and is one line."
    origin: lens:code-pattern-fit
---

The diff makes the change the decision record settles, and the cards are carried out.
Each focus question was checked and none turned up a defect.

- Finalize merges through the resolver under the parent lock for every recipe, and a conflict or overlapping-file merge goes on to the `finalize` gate. The merge record's `Overlap` is still written, and the retry after a merge-in-required error merges, gates and lands.
- The removed seams (`ConflictReroutes`, `LargeMergeLands`, `taskMerger`, the overlap note, `overlapNoteInput`, darn's `ReviewInput` override) have no reference left beyond one intended absence assertion in `finalize_merge_test.go`.
- `EscalateStopPrefix` is distinct from `LandingStopPrefix` and the decision-halt prefix, `EscalateHalt` is its only reader, and `HandsBack` stays false for it. Pawl keeps `LandingStopPrefix` and a way forward that names no `lym loom decision add`.
- `JudgesFirst(producer, reason)` reads Reason for a stop and Error for a status-built stop, and other Parent-Align and Finalize stops still meet the tackler first.
- `restep_hold` is computed beside `hand_back`, false when no check is told, and batten's `restepSkips` skips an escalate halt while leaving a pawl or other stop as before.
- Every other landing halt keeps `LandingStopPrefix`, the `on_reroute` edges change only in comments, and the declared `PATTERN-shed-stop-chain` text landed exactly.

### [NIT] haltReason doc contradicts the escalate prefix

**Location:** internal/landingshed/fixerrun.go:298
**Issue:** The doc's first sentence says each reason opens with `LandingStopPrefix`, and the lines after it give the escalate exception, so the opening sentence is now wrong for a loom or darn escalate.

## Rejected

- Test overlap in `wire_seams_test.go` (lens:code-tests): the added assertion checks the wiring into `InnerRun.HandBack`, which `TestRestepSkips` does not cover.
- Test overlap in the `handback_test.go` and `arbiterstop_test.go` rows (lens:code-tests): the table rows pin the predicate at its source, and the halt tests assert the halt's prefix; neither is redundant.
- `landingshed/doc.go:114-117` lacking the escalate exception (lens:code-affected-set): the next paragraph states it, and the sentence stays true for the halts it names.
- Orch stencil `Landing halts` wording (lens:code-affected-set): the lens itself found it consistent.
