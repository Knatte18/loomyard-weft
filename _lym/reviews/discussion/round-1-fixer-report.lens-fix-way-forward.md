```yaml
lens: fix-way-forward
outcome: reported
```

Task: check that every refusal, error, next step and criterion the fix introduced can be reached from the state it names and leads to progress.

Introduced ways forward checked:
- D4 findings-file refusals, which name the file's shape: reachable, progress.
- D4 answer refusals for a mismatched `--ask`, `not-sent` and the other end states, which name `lym loom decision add`: reachable, progress.
- D4 guard admitting the stored strand: see finding 2.
- D3 `pending-post` repost on restart: reachable, progress.
- D5 Go closing an open ask when the table ends: reachable, progress.
- D5 no-room outcome `not-sent`: see finding 1.
- D2 new criteria (goto back to Discussion-Write then `continue`, key turned off before a restart) and D3 criteria (clear after `done`, dead orch ending in `timeout`): each reachable from the state it names.

Findings:

1. LOW. D5 "A room of zero or less sends no ask … `not-sent` with that reason", read with D2's asking rule.
   - Summary: the no-room outcome names no way forward. Raising the timeout does not bring the ask back within the same seed.
   - Evidence: D5's Warn only "nam[es] both keys", while `CapAssistTimeout`'s no-room reason ends "raise review_timeout_min" (`internal/loomengine/review.go:72`). D2's rules ask a later round of the same seed only after `done` with findings, so after `not-sent` the operator raising `review_timeout_min` (or `run_timeout_min` when it is 0) gets no parent ask until a new seed or a Discussion-Write run. The step the reason points to makes no progress in the current perch, and the record does not say so.

2. LOW. D3 "writes the ask record in state `pending-post` … then posts … the ask record stores that strand", read with D4 "answers an ask in state `pending-post` or `open`" and "also admits the orch strand the ask record stores".
   - Summary: an answer to a `pending-post` ask has no stored strand for the guard to admit.
   - Evidence: the strand is known only after the post resolves the destination, but `pending-post` is written before the post. A crash between post and `open` leaves no stored strand. An orch that received the mail but no longer resolves as the parent is then refused, although D4 admits `pending-post` answers precisely for this crash. Resolving the destination and storing it in the `pending-post` write would close the gap.

No further tool calls made.
