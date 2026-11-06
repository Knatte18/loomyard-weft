```yaml
lens: fix-consistency
outcome: reported
```

Task: fix-consistency check of this round's change set against its sibling cards, decisions and rows.

**1. LOW: card 4's new sentence is looser than the rule it explains.**
- **Changed text:** `_lym/plan/04-hub-directory.md`, "a live daemon whose last record failed records the hub at the next verb it serves".
- **Summary:** "the next verb it serves" disagrees with card 7, because `deliver` and `confirm` are served verbs that never record.
- **Evidence:**
  - Card 7 says "`daemon.serve` sends the two verbs to `serveHubCall` before `resolve` ... and these verbs skip `recordHub`".
  - The sentence just before it in card 4 has the correct scope: "after each locally resolved verb", "the next such verb".
  - The later clause of the same sentence, and the `00-overview.md` decision ("served local verb"), also keep that scope.
- **Change needed:** "the next verb it serves" should read "the next locally resolved verb".

I found no other disagreements:
- **Cards 2, 6 and 10:** none mentions `recordHub`, `hubRecordInterval`, `listenHubFake`, `TestDaemon_CrossHubDeliver` or `storeInboundLocked`. Card 10's edit of the creel bullet touches a different sentence from card 5's verb list.
- **Reason codes:** the only non-review, non-lens file under `_lym` that holds them is card 8, so the new `bad-answer` code has no twin list to update.
- **Sibling text read in context:**
  - Card 8's line "A reason creel wrote as a code is not quoted" agrees with `bad-answer`.
  - Card 9's "the row below writes" still points at the `TestScheduler_Outbound` row.
  - The `00-overview.md` decision matches card 4's interval rule.
  - The `doc.go:231` claim on coded refusals still holds under the `daemonRefusal` wrap.

Tool calls:
- One grep over cards 2, 6 and 10 and the `_lym` records, because my context did not hold those cards or show where the reason codes appear outside the cards it held.
