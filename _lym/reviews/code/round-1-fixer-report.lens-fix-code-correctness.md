```yaml
lens: fix-code-correctness
outcome: reported
```

Task: check the correctness of the code the fix changed (fix-code-correctness lens).

no findings

- **New rows `BouncerRowGatesAFirstRaisedKeyOnlyWithHardenRound`:** `layoutBouncerCirclingRounds` leaves round 1 judged and round 2 reviewed, so the one `Call` makes round 2's judge spawn, and `GotSpec.Prompt` holds that decision rule.
  - With `FirstRaisedGates` set to false, the `harden_round` row failed and showed the lighter clause. With it restored, both rows passed. Deleting the line has the same effect as setting it to false.
  - The non-harden row asserts that the clause is present, so a prompt without the decision rule cannot pass both rows.
- **`focusReview` → `offChangesReview`:** the field, both table rows and the `if` were all renamed. `TestValidateReviewVerb_RoundChecks` compiled and passed at tier 1.
- **Failure message and stencil comment:** the message in `entries_burlerhalves_test.go` is a string-only change. The stencil edit stays inside the HTML comment and adds no `{{...}}` marker, and the `contracts` and `shedadapters` tests passed.

I made no further tool calls.
