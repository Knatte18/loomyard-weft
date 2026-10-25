```yaml
lens: fix-consistency
outcome: reported
```

Task: check that the round-2 fix (gate_halt fields in card 2, steps 7 and 10 and the `TestDriftVerify` rows in card 11, and the new `UpdateScope` Uses entry) still agrees with the sibling text around it.

Findings:

- **LOW**: Card 11, step 7. The fix dropped "past any `DecisionHaltPrefix`" from the old resume test, and that phrase was the only text that said the prefix comes before a halt's opening text.
  Evidence: line 62 says every `Drift-Verify` Stuck under `full` "has its reason opened with `DecisionHaltPrefix`", and step 10 says the exhausted-entry Stuck "has its reason opened with that entry's halt opening text from step 7 ahead of the seat producer's gate reason".
  Both sentences claim the reason's opening.
  Nothing parses the reason any more, so this is a wording conflict that an operator or an implementer might notice, and it ships no wrong behaviour.
  The fix would be one clause in step 7 or step 10 saying the opening text follows the prefix whenever the prefix applies.

No other conflicts:
- Step 7's resume test repeats step 3's history test word for word.
- Step 10 now writes both fields, matching step 7's "Each of this step's halts, and step 10's exhausted-entry Stucks".
- The `TestDriftVerify` rows match the step: the gate clears both fields before it runs, a failure writes them again, and the reroute, older-entry and first-entry rows run no gate.
- Card 2 lists the new fields beside `halt_named`.
- Card 7 and card 10 write the scope record through `UpdateScope`, so neither drops the new keys.
- Card 0's overview Decision (line 51), card 12 and card 14 never mention the old reason test, so none of them was left stale.

Tool calls:
1. grep of the overview and sibling cards for history, resume and scope-record mentions: I needed to know whether any sibling repeats the old reason test, and my context held only cards 2 and 11.
2. grep of cards 7, 9, 10 and 12 for scope-record rewrites and clears: I needed to know whether a verb or row rewrites the record and would drop the new fields, and my context didn't hold those cards.
