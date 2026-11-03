```yaml
lens: fix-consistency
outcome: reported
```

Task: check the fix set for places where the changed text disagrees with its surroundings, or where a twin was left as it was.

No tool calls were needed.

1. **MEDIUM** — `contracts/specs/loom-plan-spec.md:226`, "A card id never needs that, since both verbs always rewrite it".
   - **Problem:** a card id is not always rewritten. A card id naming the removed card stays as it is.
   - **Evidence:** the line above says the verbs rewrite "every card id naming a moved card", and the removed card is not a moved card. `TestRenumber_RewritesCardIDs` "Remove" expects `09-beta.md` and `9-beta` to stay untouched. The same sentence just said a removal can leave a reference to the removed card unrewritten.
   - **Why it matters:** the reviewer's proposed wording carried the same overstatement, and the fix copied it.
   - **Suggested wording:** "A card id is never reported: both verbs rewrite one naming a moved card, and one naming the removed card is left unreported."

2. **LOW** — `internal/planparser/renumber_test.go`, the doc comment on `newRenumberFixture`.
   - **Problem:** it says the fixture has "card references in the framing, a plan section and gamma's intent". The fixture now also has the Spans and Pair range lines in the plan section, and a bare reference in delta's Card Index summary.
   - **Why it matters:** a reader would not know that the Card Index line and the two range lines are deliberate.

3. **LOW** — `internal/planglyph/index_test.go`, the doc comment on `TestIndex_AppendsPlanChecks`.
   - **Problem:** it still ends "A test file the repository holds with no symbol is admitted, and one with a member is not."
   - **Evidence:** the three new rows (missing file, syntax error, same base name in another directory) pin a refusal for a file quarry cannot list or read whole. The comment does not cover them.

I checked three other pairs and they agree:
- **Mentions:** the new `insertSecondMentions`, `insertThirdMentions` and `removeSecondMentions` match the order of lines in the overview.
- **Remove-last row:** it needs no change.
- **Delta's Intent:** it is set explicitly, so the card file does not copy the Summary.

Calls made: none.
