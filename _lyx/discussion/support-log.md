# Support log — rename-target-checks

## Interview

A darn run holds no interview: the writer read the board entry and the code and made every call itself.

## Rejected alternatives

- Making the evidence-tier `rename-candidate` finding blocking inside DetectDrift: it fires only when quarry offers a candidate, so a plain removal of a later card's target would still slip through, and DetectDrift has already rewritten the plan by the time it reports.
- Failing the batch terminal and routing it to recover-batch: the cold recovery strand lacks the fork's context, which is the point of the board entry.
- Refusing on every record-batch call until the later card matches: a fork that cannot be reached, such as after a resume, would wedge the batch with no way forward.
- Computing a second delta in RecordBatch before postBatchChecks: it doubles the quarry call, and the check must read the plan after the C# re-sign rebind anyway.
- Attributing the plan finding to the card that edits the test: the fix, a Rename pair, lands on the renaming card.

## Review rounds

_No rounds yet._

## Question ledger

- Q: Does "a departure" mean a `Pattern-Departure:` trailer? Self-pick: no; it means the batch is handed back to the fork with the finding. Resolved.
- Q: Do a later card's `Uses:` refs count as targets? Self-pick: yes, as in drift detection's reference index. Resolved.
- Q: Should recover-batch refuse too? Self-pick: no; a recovery strand has ended when its report is read. Resolved.
- Q: Which languages does the plan check cover? Self-pick: Go only, by Go's test naming convention. Resolved.
