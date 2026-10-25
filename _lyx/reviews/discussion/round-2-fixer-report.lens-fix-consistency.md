```yaml
lens: fix-consistency
outcome: reported
```

Task: check whether the round-2 fix set agrees with the sibling decisions, criteria and cross-references it touches, and find any twin left unchanged.

Checked: D8's brief, hard-cap, carry-over, re-entry and bound bullets; criteria 216, 219, 220 (with its new sibling), 222 and 225; D10's no-parent and bound lines; the Notes; and the support log's rejected alternatives.

- LOW: D8's second bullet, "still applies the judgment refusal before it acts, at the harvest and on a replay alike".
  The Notes section was not updated to match, so this twin is stale.
  The new text changes the replay rule: today a replay over a round with an escalation record or decision file settles without the refusal.
  That rule is `standsAsRecorded` at `internal/shedadapters/bouncer.go:494`, and its doc says "Such a round's judgment is what the parent's decision settles over, so a replay never refuses it".
  The Notes for the plan writer still name only `settledGeneration` as "the re-entry rule D8 changes", so a plan writer who goes by the Notes misses the second rule D8 now overturns.
  Evidence: `_lyx/discussion/decision-record.md`, the Notes' second bullet, against the D8 sentence quoted above.

Everything else agrees.
- Criterion 216's "without a re-judge" is about the escalation, which still runs before the refusal.
- Criterion 219's continue and accept hold over a judgment the refusal admits.
- The hard-cap bullet's `retireContinue` path agrees with the new snapshot file, which `retireContinue` does not move.
- "One escalation per judged round" holds, because the re-judge keeps the escalation record.
- D10's line 150 ("reaches nobody until `writer_timeout_min`") agrees with the rewritten bound and its timeout halt.

No further tool calls made.
