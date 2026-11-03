```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the reviewer's F1 and F2 proposals, both fixed as proposed.

**F1 (`rewriteCardMentions` hyphen skip, `internal/planparser/renumber.go`, `renumber_test.go` fixture).**
No findings. I tried these cases:
- **`--card 3` flag, `pre-card 3`:** both are now skipped and unreported, which is the intended outcome of the hyphen skip.
- **Multibyte dashes before `card`:** an em or en dash ends in a byte of 0x80 or above, never `'-'`, so `—card 3` is still rewritten.
- **Offsets:** `masked` has the same length as `line`, so `masked[span[0]-1]` is the same byte in both.
- **Card ids such as `--card 9-beta`:** they are blanked in `masked`, so `\d+` cannot match them and they are untouched by the hyphen check.
- **Test regression power:** the fixture line `insert-card {first+2} slug` sits in the framing. On an insert or remove at offset 1, card `first+2` moves, so the old code would rewrite it and `framedAfter` would differ from `prose.Replace(framedBefore)`. The test fails with the defect in place.
- **Prose replacer:** no key in `insertSecondProse` or the other prose maps occurs inside the new line, so the replacer does not touch it.

**F2 (`contracts/specs/loom-plan-spec.md:227`).**

- **[MEDIUM]** Changed text: "A card id never needs a hand fix: both verbs rewrite one naming a moved card and leave one naming the removed card, so a card refers to another card by its card id."
  - **Summary:** the sentence says a card id never needs a hand fix, but the same sentence says an id naming the removed card is left as it is.
  - **Evidence:** the verbs leave an id naming the removed card, per the sentence and the `TestRenumber_RewritesCardIDs` comment "a removed card's id stay". That id is neither rewritten nor listed in `prose_mentions`, which holds bare references only. After `remove-card` it points at a card that no longer exists, so a planner must repair it by hand. The two halves of the sentence contradict each other. The trailing "so a card refers to another card by its card id" has no clear antecedent and does not justify the claim.
  - **Fix I'd propose:** state the behaviour without a claim about hand fixes, for example "A card id naming a moved card is rewritten and is not reported under `prose_mentions`; one naming the removed card is left as it is and is not reported."

Tool calls: none; the context already held every file this lens needed.
