```yaml
lens: fix-consistency
outcome: reported
```

Task: check that the fixer's change set agrees with the cards, rows and cross-references around it (fix-consistency lens).

**Finding 1: MEDIUM**
- **Changed text:** `_lym/plan/12-outbound-routing.md`, Intent: "The stale line (`reportStale`) and the dropped-message line (`refused`) post through `postTo`'s recovery, and a refusal of either counts toward no message's `refusalCap`."
- **Summary:** The new sentence disagrees with the parts of the same card that it depends on, and those parts were not changed to match.
- **Evidence:**
  - The Create glyph is still `postTo(ctx, record *outboxRecord, message creelengine.Message, orch, text string, sender Sender) error`. That signature needs a message to count against. The stale line has no message, and neither line has a `Sender`.
  - The `postTo` paragraph still says "Each post Slack refuses with a code counts once toward `refusalCap` through `refused`".
  - `refused` itself posts the dropped-message line. If that line goes through `postTo`, and `postTo` counts through `refused`, the card describes a loop. Nothing in the card says `postTo` gets a mode that does not count, or a sender for a line.
  - The fix changed one part of the `postTo` contract and left its twin, the signature and the counting sentence, as they were.

No other disagreement found:
- After the renumber, the references between card 5 and card 6 ("card 5 adds", "Until card 6", "card 6's login") point at the right cards.
- `ReadCredentials` and `WriteCredentials` in card 13 now resolve to card 5. `PostAs` and `OpenConnection` exist in cards 1 and 2.
- The keepers and harnesses that F4's nil-`LiveHubs` panic reaches all set the seam: `keeperHarness`, the inbound `newHarness`, `newOutboundHarness` and `newRelayRun`.
- The holder rows in card 9 match its Intent.
- The `MissingScopeNamesTheScopes` row is split consistently between card 6 and card 7.
- The Card Index and the overview prose match the new card.

Tool calls:
- One grep over `_lym/plan` for `OpenConnection`, `PostAs`, the credential symbols, `postTo` and `refused`, and the card references in `lens-review.md`. My context had cards 1, 2, 8 and 13 only in part or in their old form, and did not have the card mentions in `lens-review.md`.
