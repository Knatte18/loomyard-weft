```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the reviewer's proposals the fixer carried out, under the fix-proposal lens.

1. **MEDIUM**: D13's decision-notice bullet (F5) now says "holds none of them back: a halted run's live Weaver strand gets the notice". This wakes a strand while the arbiter edits the tree.
   - Under the new D6 text, the row stops a failed attempt's strand before it returns Stuck. So the only way a halted run still has a live Weaver strand is a crash or interrupt that left it running, for example a gated strand relaunched and sitting idle with its report already written (D12's rationale).
   - While the run is halted, the stop chain runs. The arbiter "may edit and commit" (ab5d49e0; D7 says it "commits and then names a goto target").
   - The notice now wakes the idle strand to act on the decision. Two agents then edit one tree with no row owning the strand. That is the race D12's rationale names ("would race two agents on one tree").
   - `heldBackReason`'s hold-back prevented exactly this. The proposal assumes D12 re-attaches before anything else acts, but re-attach happens only at the next row step, after the arbiter has finished.

2. **MEDIUM**: The D1 guard's way forward and D2's `lym weaver reset` (F9) lead to Webster rebuilding work Weaver already landed.
   - Case: Weaver records batches 1–2 done and commits them. The operator flips to `webster` and runs `lym weaver reset`, which archives `_lym/weaver/`.
   - D13's seam now finds no begun batches anywhere and falls back to the setting. The plan gate no longer freezes cards 1–N, and Webster partitions the whole plan, rebuilding the landed cards over their own commits.
   - This contradicts D1's "A run never switches implementer with work half built" and D13's rationale ("or the plan is rewritten under landed work").
   - The proposal names the reset as a way forward without saying what happens to the branch's landed batches.

3. **LOW**: D8's journal append keyed by heading (F13), together with D6's same-rung retry and D12's "report path, unique to the batch and attempt" (F1, F14).
   - The record never says whether a same-rung retry after a machinery loss, or D7's re-entry attempt, gets a new attempt number.
   - If a retry reuses the attempt number, two problems follow:
     - D8's "skipped when the journal already holds it" drops the retry's journal entry once the lost attempt's heading exists.
     - D12's identity collides with the lost run's report path.
   - The keying is sound only if every attempt, retries and re-entries included, has a distinct number, and the record doesn't state that.

4. **LOW**: D13's refusal "when ... either state file fails to load" (F3) applies to every reader the seam serves.
   - Those readers include the decision-notice guard and `lym fabric merge-in`'s in-flight warning.
   - Case: Webster's state file is left corrupt from an earlier abandoned attempt while Weaver runs. `lym loom decision add` notices and merge-in's warning then refuse, even though the corrupt file is for the implementer not in use.
   - The proposal sets one refusal policy for the guard and the plan gate, where refusing is right, and for advisory readers, where it blocks unrelated work.

No further tool calls made.
