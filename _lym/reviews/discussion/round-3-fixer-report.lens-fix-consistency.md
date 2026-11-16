```yaml
lens: fix-consistency
outcome: reported
```

Task: check that the round-3 change set agrees with the sibling decisions, criteria and cross-references it touches (fix-consistency lens).

**MEDIUM: the D12 acceptance criterion still attaches a guid-less fixer that D12 now stops**
- Changed text: `decision-record.md`, the crash/resume acceptance criterion (D12). Its fixer clause was rewritten, but its opening clause was not.
- Summary: the opening still says the call "attaches through the recorded output file to a live run of a batch attempt or fixer strand recorded in flight, even when state never recorded its guid". The new D12 says something else:
  - only batch attempts attach through `AttachGated`;
  - a live fixer attaches through `gatefixer.SpawnOrAttach`;
  - a fixer number with no `gatefixer` record is probed through `ProbeGated`, its strand is stopped, and it counts as a machinery loss.
- Evidence: the new D12 line "Then, for every batch attempt state records in flight, it attaches through `AttachGated`…" and the new line "A fixer number state records with no `gatefixer` record … stops a strand it finds through the handle's `Stop`". The same criterion's newly added fixer clause agrees with D12, so the opening clause contradicts it.

**LOW: decision delivery to a fixer reattached through `gatefixer` is no longer stated**
- Changed text: D12's new fixer lines.
- Summary: D12's decision-delivery sentence covers only a strand attached through `AttachGated`. The fixer lines now sit after it, so a fixer reattached through `gatefixer.SpawnOrAttach` gets no statement that it receives the decisions recorded meanwhile.
- Evidence: D12 says "Before it awaits an attached strand, Weaver sends it every decision entry recorded since the attempt began…", and that sentence comes before the fixer lines. D13's decision-notice bullet still says "D12's attach sends a strand still in flight the decisions recorded and not delivered meanwhile", and D13 counts the fixer strand role as an implementer strand. D9 still records "the decision entries delivered to its strand".

**LOW: two support-log history lines were not updated alongside Q9, Q15, Q16 and Q17**
- Changed text: `support-log.md`, where Q9 and Q15 were revised so the fixer carries no gates, and Q16 and Q17 were marked answered.
- Summary: two Turn 4 lens-review lines were left as they were. Other history lines in that list carry a "(revised in review round N…)" note, and these two have none:
  - Turn 4 still says "fixed: D4 and D10 attach them to every batch and fixer strand".
  - Turn 4 still says the board-scope push finding was "kept as an Open risk, rephrased as a question for the parent". Open risks now reads "None.".
- Evidence: `support-log.md` Turn 4, the "walkthrough + bound-breaker + interactions (MEDIUM/LOW): the row's `gates` entries had no runner" bullet and the "board-scope (MEDIUM, decision)" bullet.

I made no further tool calls.
