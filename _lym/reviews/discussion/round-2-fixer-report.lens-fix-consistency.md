```yaml
lens: fix-consistency
outcome: reported
```

Task: check, through the fix-consistency lens, that the round-2 change set agrees with the surrounding decisions and criteria.

Findings:

1. **LOW.** Concerns D8 lines 158 and 181, "after the channel's cursor" and "Per message after the cursor". D4 now gives a bridge channel a delivery cursor, an operator cursor and thread cursors, but these two sentences still say "the cursor". The per-message list is read after the delivery cursor, and its operator bullet uses the operator cursor, so "the cursor" no longer names one thing. Evidence: D4 line 81 lists "delivery cursor, operator cursor, thread cursors and thread floor".

2. **LOW.** Concerns the new D8 bullet that skips an envelope message older than creel's journal retention, set against untouched sibling text. D8 still says "an approval delivers every held message in order", and its Bound still says "a held message is delivered only by an approval or the button". Under the new bullet, a held unpinned message older than seven days is skipped when it is reread after the approval. That contradicts "every held message".

3. **LOW.** Concerns D7's new sentence "The relay counts a message's Slack refusals in its bridge-post record". The surrounding text still describes that record as holding only accepted messages:
   - "keeps each bridge message Slack accepted, with its Slack timestamp, in a bridge-post record";
   - "An entry is removed only once creel has answered its settle".

   A refusal-count entry belongs to a message that was never accepted. If creel returns that message at its hour, the relay never settles it, so no stated rule removes the entry. D3 was narrowed to "every accepted message", but its twin in D7, the removal rule, was left as it was.

4. **LOW.** Concerns the narrowed resync test in D8, which now reads only channels with the bridge name's shape, with a `--` between two ends. Three untouched sentences still say "`bridge-` channel": D8 line 172, and the criteria at lines 301 and 305. They now describe a set wider than the one the resync reads. The D8 Bound at line 191 was updated to "bridge-shaped channels", so these are its twins left as they were.

Tool calls:
- One grep of `decision-record.md` and `support-log.md` for the changed terms (peer app, read cursor, thread floor, "the cursor", `bridge-` channel). My context held only my own edits, not the sentences around them that still use the old terms.
