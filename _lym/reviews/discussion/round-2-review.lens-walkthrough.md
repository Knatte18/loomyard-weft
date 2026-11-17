```yaml
lens: walkthrough
outcome: reported
```

Task: run the walkthrough lens over the round-2 decision record, executing one concrete case per decision.

**Walkthrough list**

- **D1** (one case per kind; purpose met):
  - A routed round with design finding F1 and decision finding F2: the fixer writes `F1 fixed`, and F2 is optional or takes a `ruling`. The fix check passes and D2 reads the result.
  - A review holding only F2: `skipFix` writes `skippedRoutedFixerReport` with empty dispositions and `spawned: false`, which D6 counts as changed.
- **D2** (purpose missed): round 1 holds decision findings F1 and F2, and the Bouncer escalates with cause `decision`. The parent runs `lym loom decision add` for F1 only, then `circling continue`, and round 2 renders.
  - The block's first answer source, a settled decision file of cause `decision` for round N-1, never reaches a round N.
  - Its second source, the added entries, is not tied to a finding. See F-A.
- **D3** (purpose missed for an unanswered decision finding): a round-2 reviewer handles a disputed F1 (withdraw, or `open` with `answer` and `carries`) and an answered decision finding F2. The disputed path holds. The decision path breaks through D2: the gate forces an unanswered finding to `resolved`. See F-A.
- **D4** (purpose met): in a source-scope incremental round 2, the changes file runs from round 1's start-of-review working-tree id to round 2's start, which is exactly fixer 1's edits, committed or not. A new BLOCKING finding on a touched file passes, and a non-BLOCKING one is refused.
- **D5** (purpose met): in an overlay Discussion-Review, round N's hash is taken before the reviewer starts, the fixer edits `decision-record.md`, and the post-fix hash differs. A re-run review keeps the first record.
- **D6**, two cases:
  - Purpose met: with `review_stalemate_rounds` at 2, round 1 is all disputed and unchanged with a CONTINUE verdict, giving a count of 1. Round 2 carries F1 open, is all disputed and unchanged, and reaches 2, so the round escalates with cause `stalemate`. A `continue` restarts the count after round 2.
  - Gap: a round that `routeDecisions` takes. See F-B.

**Findings**

1. **F-A**
   - Severity and class: BLOCKING, design.
   - Location: `_lym/discussion/decision-record.md` D2, second paragraph ("Beside each `decision`-class finding the parent held…"), and D3's answered-decision bullet.
   - Summary: D2's parent-answer channel cannot say which finding was answered, and one of its two sources never fires. D3 then forces every held decision finding to `resolved`, so a question the parent never answered drops out of the review.
   - Evidence: `internal/shedadapters/circling.go:232`; `internal/shedadapters/bouncerdecision.go:173-174`; `internal/shedadapters/bouncer.go:1313-1330`; `bouncerdecision.go:354-373`.
   - Walkthrough:
     - A decision file is `settled` only on an accept (`circling.go:232`).
     - An admitted accept of a decision escalation goes through `acceptRound` (`bouncerdecision.go:173-174`), which returns `shedengine.Done` (`bouncer.go:1313-1330`). The segment ends, so no round N reads round N-1's settled decision file.
     - The added entries come from `addedDecisionTitles`, a record-wide list with no link to a finding id (`bouncerdecision.go:354-373`). D2 renders them beside every held finding.
     - D3 makes the round-check entry refuse any status but `resolved` for "such a finding". With F1 answered and F2 unanswered, the gate forces F2 to `resolved`, and it is never escalated again. That contradicts D2's own "shown as still with the parent" and D3's purpose.
     - Any unrelated added entry, such as a PATTERN change, has the same effect.
   - Fix:
     - Drop the decision-file source from D2.
     - Render the added entries past the count once in the block, not as a per-finding answer.
     - Have the reviewer mark each held decision finding `resolved` with an `answer` naming the entry title that settles it, or `open` and carried.
     - Have the round-check entry refuse `resolved` on a held decision finding only when its `answer` names no added entry from the block.
     - Update D2's and D3's acceptance criteria to match.
2. **F-B**
   - Severity and class: LOW, consistency.
   - Location: `_lym/discussion/decision-record.md` D6, first paragraph, and the placement paragraph.
   - Summary: the post-fix target id is recorded "the first time the Bouncer settles round N", but the check sits only in `settleUndecided`. A round that `routeDecisions` settles is not one the record says gets its id recorded, and a round with a snapshot record but no post-fix id is classified neither changed nor unchanged.
   - Evidence: `internal/shedadapters/bouncer.go:1037-1044`, where `routeDecisions` returns before `settle` and `settleUndecided`.
   - Walkthrough: round k is decision-escalated, then continued, and the fixer changed nothing. When round k+1 counts the run of unchanged rounds, round k's post-fix id is either missing or computed late, after the parent's `lym loom decision add` edited the overlay target. The count's input then differs from what the fixer left.
   - Fix: state that the post-fix id is recorded at the top of `settleUnlessRefused`, before `routeDecisions`, so every settle path records it. Also state that a round with a snapshot record and no post-fix id counts as changed.

Tool calls:
1. Grepped `lymcwd` doc for warp and the `burlerengine` geometry, to check whether D4's worktree-relative paths match the burler worktree root. The manifest lacked the geometry; the file was missing.
2. Grepped `lymcwd` for warp/weft and the loomcli `WorktreeRoot:` wiring, for the same question. No mismatch was established, so no finding.
