```yaml
lens: fix-consistency
outcome: reported
```

Task: check that the round-3 fix to the decision record and support log agrees with the sibling decisions, criteria and cross-references it touches or depends on.

**MEDIUM: Notice marks have no home for holds on a channel absent from the bridge map**
- Changed text: D4's bridge-map fields, "notice marks: one per kind of relay notice of D4, D7 and D8, … so a restarted relay posts no second notice for an episode". Also the bridge-notice record, which is kept only "for an unpinned creator or a retired orch (D8)". Also the new D4 criterion, "a relay restarted during a hold episode posts no second notice for it".
- Summary: several of the notice kinds this text covers belong to holds whose channel has no bridge-map entry, and the bridge-notice record does not cover them either. A restarted relay therefore has nowhere to read their marks from.
- Evidence:
  - D4 holds `bridge-name-taken` when the channel was "made by anyone else" and `bridge-archived` when "the listing finds [it] archived". In both cases the relay does not adopt the channel, so it never enters the bridge map.
  - D7's `peer-unapproved` notice is posted "in the sending orch's channel", and it can be posted before any bridge channel is created. D7 creates the channel only when posting, and a held message is never posted.
  - The bridge-notice record lists only D8's two unmapped cases.
  - The new D4 restart criterion therefore cannot be met for the `bridge-name-taken` and `bridge-archived` holds that D4 itself names. D7's criterion "posts one notice per episode" has the same gap across a restart.

**LOW: Dropping a channel from the bridge map silently clears its notice marks**
- Changed text: D4's bridge-map fields, which now hold the notice marks.
- Summary: the marks now live on the map entry, but nothing says whether dropping the entry also ends their episodes.
- Evidence: D4 still says "A post refused because the channel is archived or gone drops the channel from the bridge map". If that channel is later found archived, it holds as `bridge-archived`, a D4 hold whose mark has no home (see the first finding). The fix leaves that sibling sentence unchanged and does not say the drop ends the episode.

Nothing else is stale:
- **D3 against D7 and D10:** D7's "creel returns the message at its hour", D7's take-again rule, D10's poll and the D3 criterion "no per-hub outbound worker starts … at the scheduler's start or on a post" all agree with the rewritten D3. The Notes line on the hour timer agrees with D3 as well.
- **Constraints:** both the `PATTERN-tolerant-records` bullet and the `PATTERN-lym-files` bullet were updated for the bridge-notice record.
- **Archived-channel way forward:** the D4 text, its criterion and the support log agree.
- **Approval and cursor:** the D8 sentence on the cursor after an approval agrees with D8's "an approval delivers, in order, every held message" and with the new criterion.

Further tool calls: none.
