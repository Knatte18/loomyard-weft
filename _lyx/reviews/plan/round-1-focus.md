---
round: 1
focus:
  - "Card 2, persistTransient: settle whether the new 'this write recorded a verdict' test (history longer, or equal non-empty length with the last entry's Repeats differing from the re-read file) separates a fold from every no-verdict write that reaches persistTransient (pause writes, the step-3b resume write, the producer-error and cancellation arms), and whether step_fold_test.go exercises a no-verdict write landing right after a fold."
  - "Card 2, step_fold_test.go: settle whether the 'exempt Stuck, Awaiting, exempt Stuck' case can be driven through Step as written, given an Awaiting verdict halts the run, and whether the card says how the run gets from the Awaiting halt to the third step."
  - "Card 2 ImpactSummary: settle whether 'counted Stucks, routing and budget decisions are unchanged' is a real blast-radius conclusion once readers of history length are counted: internal/loomengine/anomaly.go (HistoryLength, countDoneEntries), battencli's recentHistory truncation, and ly-drive's history_length (D10)."
  - "Cards 3 and 4 against D7-D9: settle whether the not-yet-re-stepped reason names both D7 remedies and omits the re-approve hint, whether card 4's tests cover every D7-D9 acceptance criterion (new identity resumes regardless of the recorded length, old-layout marker, a longer child history), and whether card 3's rule that an unparseable second line reads as the old layout is licensed by D8 or adds scope beyond it."
  - "Decision-record coverage sweep: settle whether every Decision D1-D10 and every Constraint (single persist per step with the read slice unmutated, Told-Geometry marker path, ReadJSONStrict fields on HistoryEntry, docs in the same commit) is carried by a card, including the HistoryEntry and persist doc-comment updates the record's Notes call for."
---

Output on a budget-exempt Stuck is `output.Path`, which battenshed leaves empty, and the reason text is not persisted in `HistoryEntry`.
So the driver-exit grace reason's changing elapsed time does not defeat the fold key.
That was checked and is not a focus entry.

Card 3 standing as its own card is consistent with the rubric's granularity rule: it holds pure helpers with their own tests.
So card 3 is not flagged for granularity.
