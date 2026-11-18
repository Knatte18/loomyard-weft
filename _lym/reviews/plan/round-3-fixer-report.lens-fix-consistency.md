```yaml
lens: fix-consistency
outcome: reported
```

Lens: fix-consistency. I checked whether the F17 change set to card 14 agrees with its sibling text, rows and cards, and whether any twin was left unchanged.

- **LOW**: the existing Wait-path stop-race row in card 14 ("a `StopStrand` run from `fakeReed`'s `OnEndStrandProcess` hook, between the mark and step 4, leaves the record stopped and no strand up") was not updated to match the new contract.
  - **Problem:** the record-write paragraph now says a stop-ended relaunch returns a Result whose Outcome is `OutcomeDied`, beside nil on the `Wait` path. The new before-mark row and the new `start` row both assert that outcome. This Wait-path twin still asserts only the record and the strand, so nothing pins the returned outcome or the nil error on the `Wait` path.
  - **Evidence:** card 14 row "a `StopStrand` run from `fakeReed`'s `OnEndStrandProcess` hook, between the mark and step 4, leaves the record stopped and no strand up". Compare the new sentence "on the `Wait` path beside nil, and on the start path as the load-turn paragraph states".

Everything else agrees. Card 16:32 ("It returns as card 14's start path returns: a relaunch that ends the run hands `start` an error wrapping `ErrNotStarted`…") still holds under the new stop-ended start-path return. Step 1's parenthetical ("ends as a stop-ended relaunch, with no reed op") matches the record-write paragraph's before-mark sentence and the edited row. The load-turn paragraph defers strand removal to that paragraph, so the two do not conflict. The only other occurrences of "the stop's outcome" are in old lens-review files under `_lym/plan/`, which are not part of the plan cards.

Tool calls:
1. Grepped `_lym/plan/` for "stop's outcome", "stop-ended", "startupStrandRemoved", and card 16's stop and `ErrNotStarted` sentences. My context did not have card 16 or the other plan cards, which might repeat the changed wording.
