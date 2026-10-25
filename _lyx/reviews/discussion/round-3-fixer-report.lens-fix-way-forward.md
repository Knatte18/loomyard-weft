```yaml
lens: fix-way-forward
outcome: reported
```

Lens fix-way-forward: checking the ways forward the fix introduced.

**[BLOCKING] Folded history entries can stop the parent's resume from ever ruling (D6, the new "An entry is the resume from that halt only when…" sentence)**

The changed text: "An entry is the resume from that halt only when the run status's last history entry is this row's own Stuck in that halt, appended at or after that time."

The problem: shed folds consecutive identical budget-exempt Stucks into one history entry. The folded entry keeps the first call's `At`, so the check can fail on every resume.

Evidence: `internal/shedengine/status.go`, in the `HistoryEntry` doc and fields:
- "A budget-exempt Stuck folds into an immediately preceding identical exempt Stuck rather than appending."
- "`At` keeps the first call's time."
- `LastAt` holds the time of the most recent folded call.
- A Drift row has no `on_stuck`, so its Stuck is exempt ("one at a row with no OnStuck").

A case that loops:
1. Under scope `full`, a failing final compare with acceptance A ends in a decision halt. The row records A at T1, and history gets a Stuck with `At` T1'.
2. The parent resumes. The row marks A `ruled`, the compare fails, a fresh agent accepts B, and the final compare fails again.
3. The row records B at T2 and returns a second decision halt. The goto and resume code (`goto.go:135`) suggests no new entry comes between the two Stucks. So the second Stuck folds into the first, with `At` still T1' and T1' &lt; T2.
4. The parent runs `lyx loom resume`. The entry does not count as the resume from the halt, so it halts again and records B at T3. That Stuck folds too.

No way forward the halt names, resume, the parent's accept or withdraw and resume, ever marks B `ruled`. The run cannot get past `Drift-Verify` without a goto past it, and `CheckGotoTarget` refuses that.

Proposed fix: compare against the entry's `LastAt` when it is set, else `At`. Or base the test on something folding cannot hide, such as a halt counter or an id the row writes into the scope record and checks against the status.

**[LOW] The history entry stores no halt reason (same sentence)**

The changed text: "this row's own Stuck in that halt".

The problem: a history entry cannot show that the Stuck was a decision halt. Only Producer, Outcome and the times can be checked.

Evidence: in `status.go`, `HistoryEntry.Reason` is "the entry gate's text on a skipped entry, empty on every other entry", and no other field holds the Stuck reason.

The sentence promises a check the status cannot provide. The rest of the test, a Stuck of this row at or after the time it recorded, still works once the fold issue above is fixed. Proposed fix: reword it to "this row's own Stuck".

**[LOW] "none" leaves out the stand-by halt's own ways forward (D5, the new giving-up sentence)**

The changed text: "and at any other `Drift-Verify` halt none, since that halt's compare can run, so only its named accept or fix lifts it".

The problem: under scope `full`, the decision halt on an `unruled` acceptance names resume to stand by it, or `lyx drift withdraw` and resume, not accept or fix. So "only its named accept or fix" misstates that halt's ways forward. Proposed fix: write "only the ways forward it names lift it".

Checked and sound:
- `lyx drift defer` at a `Drift-Await` halt, then resume, then a skipped `Drift-Verify` while the flag stays off.
- `lyx drift waive` admitted on no counted capture after the flag goes off and on again.
- The `spawnLoop` settlement.
- The compare-marker lock.
- The new acceptance criteria are reachable.

Tool calls:
- Read `internal/shedengine/status.go` lines 60–124 and grepped the history appenders, to see what a `HistoryEntry` holds and how entries are appended or folded. My context lacked the struct's fields and fold rule.
