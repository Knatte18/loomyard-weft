```yaml
lens: fix-affected-set
outcome: reported
```

fix-affected-set: I re-ran each finding's search over the plan cards to check the fix covers every member of the set, not only the ones the review named.

Unchecked findings: none.

- **F1 (NewDialer call sites):** card 2 declares the four-parameter signature. Card 6's closure now passes all four arguments. Card 13's `runRelay` call already passed all four. No other card calls `NewDialer` or `newDialer`.
- **F2 (LiveHubs-error way-forward sites):** the only notices for a failed `LiveHubs` read are on card 9's Intent and test row and card 10's Intent and test row. All four now quote the seam's error as it is, and card 6 now states that rule for `liveHubGeometries`. No card still tells the operator to move a sound `hubs.json` aside. The remaining "aside" matches (card 4 lines 48 and 60, card 7 line 56) are about the channel map, not the hubs record, so they are outside F2's set. Card 6's login row (line 98) names the seam's error as it is and stays consistent.
- **F3 (missingTokenExit branches):** bot token alone, both tokens with no setup file, and now an app-level token with no bot token each have a clause in the first `TestRunRelay` row.

no findings

Tool calls:
1. grep of cards 00–09 for hubs-record and way-forward sites, to rerun F2's search over the cards.
2. grep of cards 00–09 for `NewDialer` and `newDialer` call sites, to rerun F1's search.
3. grep of cards 10–13 for both sets, because the first two globs (`0*.md`) left out cards 10 to 13.
