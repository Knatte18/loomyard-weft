---
round: 1
focus:
  - "Card 4 (send-settles-and-confirms): in the diff, does a failed submission confirm clear only the send's own typed text, and does `ErrSubmissionNotLanded` surface on every path where the confirm window closes without a landed submit?"
  - "Cards 3 and 4 (send-context decision): do all timed waits (idle polls, appearance polls, settle reads, confirm backoff) go through the `sendContext` `Clock`, and does a zero deadline mean none, set only by `sendWithin`?"
  - "Card 5 (start-send-teardown): does every start-time send or persist failure tear the run down as not started, with no path that leaves a half-started run recorded as running?"
  - "Card 6 (gate-boundary-idle): does an unlanded gate re-prompt retry on a later tick without spending an attempt, and does a genuinely failed attempt still count?"
  - "Cards 7 to 10 (stop verb): does every lyx-initiated strand removal in burler, webster and attach record `stopped` first through the stop verb, and are the deleted removers left with no remaining callers (per-type mechanical check for the `Delete` groups)?"
  - "Test economy: for each new test, does an existing test already cover the same behavior, and does every behavior the diff adds have coverage?"
---

Focus is derived from the plan's cards only; the diff itself was not yet read.
