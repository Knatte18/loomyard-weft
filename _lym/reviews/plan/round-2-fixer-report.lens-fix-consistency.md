```yaml
lens: fix-consistency
outcome: reported
```

Lens: fix-consistency. I checked whether the round-2 change set agrees with the sibling cards, rows and cross-references it touches.

1. LOW — card 09, the "The tick's restarts are capped" bullet. The new sentence says `starter.ensure` writes `LastSpawnAt` and `NextStartAt` "before it calls `spawn`". The untouched next sentence in the same bullet still gives the reverse order: "It then spawns, unless the count reached five, and sets `NextStartAt` to the spawn time plus one minute … every spawn sets it". Read in order, that says the spawn comes first and the write after, which is the order F1 removed. The fixed text and its twin in the same bullet now disagree.

2. LOW — card 12 line 25, which the fix did not touch: "A `gave-up` outcome, card 9's cap after five relays in a row that never made their first read". After F1, a spawn that fails to start, with no relay ever existing, also counts toward the cap. The new `TestEnsureRelay` row has a fake spawn that errors five times and ends in `gave-up`. The phrase "five relays … that never made their first read" no longer covers what card 09 counts. It should say something like "five spawns in a row with no accepted first read".

Everything else agrees:
- Card 12's `TestHubDobbyDeps_PrimeTickEnsuresRelay` row still holds: `ErrSpawnUnderTest` is now returned after the state write and still matches `errors.Is`.
- Card 04's `RelayState` field list and its "written by the relay and its starter" agree with the new write order.
- The decision record's F3 entry matches card 07's new sentences and row.
- Card 08's `ReadWait` timeout references agree with card 03's reworded doc.
- Card 03's Edit list and ImpactSummary agree with its Intent.
- Card 10's `newRoot` sentence matches its existing Edit target.

Tool calls:
- grep across `_lym/plan` and the decision record for the changed identifiers: I needed sibling cards that my context lacked.
- Read of `_lym/plan/12-dobby-relay-tick.md`: the card that consumes the cap and `ErrSpawnUnderTest`, which was not in my context.
