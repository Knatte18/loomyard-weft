```yaml
lens: fix-affected-set
outcome: reported
```

Task: for each finding whose premise is a set, check the fix covers every member of that set, not only those the review named.

Unchecked: none. F1, F2, F3 and F5 were checked. F4's premise is a single guard hole, not a set.

- **F1:** card 12's Intent prescribes three Stucks: moved warp head, missing baseline directory, and no `awaited_at`. `TestDriftExpect` now has a row for each, and the prefix sentence covers all three. No member is left out.
- **F2:** I checked card 16's Intent and steps 1–10 against its `TestDriftVerify` rows. Every Stuck now has a row, including the interim `command` Stuck. Card 17 now names the interim row it replaces.
- **F3:** a grep of cards 00–18 finds no driftengine code that needs `Darn-Write` or `Webster` as a constant, other than card 16's route derivation. Card 16 now creates both constants. Card 19 lists them under Uses and now credits them to card 16.
- **F5:** every refusal in card 5's Intent has a row in `TestParseExpectation`. The rule lists are: under `lines`, the unknown case, a missing removed case, an unknown key, a malformed range and a case-level `why`; under `command`, a `keys` or `lines` entry; under both, an empty `why`, an `n` below 1 and an unknown key. Every refusal in card 6's Intent has a row in `TestParseJudgement`: the spawn id, the cell ids, the ruling set, the empty reason, the open range, the `cases`/`cells` lists, the repeated case, no case after a failing compare, and an unknown field.

no findings

Tool calls:
- One grep over plan cards 00–18 for `Darn-Write`, `Webster` and `Drift-Recheck` row-name use, plus card 12's and card 18's Stuck lines. My context had only cards 5, 6, 12, 15, 16, 17 and 19, so it couldn't show whether an earlier card also needs those constants.
