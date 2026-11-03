```yaml
lens: fix-proposal
outcome: reported
```

Task: test the reviewer's proposals the fixer carried out (F1, F2, F3, F5, F6, F7) by building cases where the proposed change goes wrong.

Proposals challenged: F1 nested expansion and exit rewrite, F2 the block's end and status "entered", F3 the goto and re-launch bounds, F5 the skip-target readers, F6 the block `on_done` refusal, F7 D8's bound.

**MEDIUM: D4's end condition misses a nested block row that is passed over**
- Changed text: decision-record.md, D4: "an inlined row whose expanded `on_done` is the block's exit returns Done, or its entry gate does not hold and its skip routes there". F1 also changed D1: "A block row's `on_done` may be empty: inside another block, expansion rewrites it… so a sub-recipe may end in a block".
- Case: P holds block B on M, and M ends in a block row C on I with an empty `on_done`.
  Under the new D2, C's expanded `on_done` is I's entry, and its skip target is rewritten to B's exit.
  When C's gate does not hold, the run reaches B's exit through C's skip target.
  D4's condition requires a row whose expanded `on_done` is the exit, and C's is I's entry, so D4 does not count this as B ending.
  The D4 acceptance bullet ("an inlined row whose expanded `on_done` is the exit returns Done or is skipped") has the same gap.
- Evidence: D2 says expansion sets the block row's `on_done` to the sub-recipe's entry and moves its own `on_done` into the skip target. D3 says a skipped row routes to the skip target when it is set.
  The two proposals combine into a route D4 does not name.

**MEDIUM: D4's reader rule treats a goto out of the block as the block's end**
- Changed text: decision-record.md, D4: "a reader finds it as the exit's first entry after the block row's, the first entry whose block path does not include the block."
- Case 1: the run halts at Discussion-Write inside `Cartoon`, and the operator moves it back to Preflight with a goto.
  The goto entry's producer is Preflight, whose block path is empty (D6), so under the reader rule it is "the first entry whose block path does not include the block".
  The reader takes the block as ended, though the exit, Drift-Baseline, was never reached.
- Case 2: when the block row is skipped (passed over), the next entry is the exit's, and the rule reads it as the end of a block that was never entered.
- So the two halves of the sentence disagree: "the exit's first entry" is not always "the first entry whose block path does not include the block".
- Evidence: goto.go:136 appends a `HistoryEntry` for the target on any admitted goto. gotoAdmitted (goto.go:338-341) admits every row before the reference row, the rows before the block row included.

Tool calls beyond context: none.
