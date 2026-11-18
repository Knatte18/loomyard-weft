```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the reviewer proposals the fixer carried out as proposed (F1, F2, F5, F6, F7, F8).

**LOW: F6's list of texts to reword skips tapestry's header.**
- **Changed text:** card 10's new Intent sentence: "Every text that says PR-Review's Stuck is always a Halt now says its Stuck is a Halt except a marked send-back's reroute to Code-Bouncer: `internal/loomrecipe/doc.go`, the header comment of `darn-recipe.yaml` and `sampler-recipe.yaml`, and the PR-Review row comment of all three recipes."
- **Summary:** The proposal names only the texts with the words "always a Halt". `contracts/recipes/tapestry-recipe.yaml:29-31` makes the same claim in other words, and once the reroute lands it is false and stays unedited.
- **Evidence:**
  - `contracts/recipes/tapestry-recipe.yaml:29-31` lists PR-Review among the rows that "carry no on_stuck ... and their Stuck halts the whole run just as a gate's does".
  - With cards 9 and 10, a marked send-back makes PR-Review return a Stuck with `Reroute` and no `Halt`, so that sentence becomes false for PR-Review.
  - The header already has the shape the fix needs, at `contracts/recipes/tapestry-recipe.yaml:39`: "Webster's Stuck halts the run for a human too, with one exception: Webster declares on_reroute: …".
  - Card 10's older sentence, "The header comments that say PR-Review has no edge out", does not cover this line either, since the line speaks of halting and not of edges.
  - Fix: add tapestry's header (lines 29-31) to the list of texts reworded, using Webster's exception sentence as the model.

I found nothing to challenge in the other proposals:
- **F1:** cards 1 and 2 never touch `driftTestEnv`, `TestDriftEntries` or `driftVerifyEntry`, so card 3's new rows do not collide with an earlier card.
- **F2:** the subtest is tapestry-only, and Webster already has a backward reroute edge there, so a reroute on PR-Review fits the shape the test pins.
- **F5, F7 and F8:** each states what the code and D2 already hold, and none contradicts a sibling card.

Tool calls:
- One grep of cards 01 and 02 for `driftTestEnv`, `TestDriftEntries`, `driftVerifyEntry` and `requires`, to rule out a sibling card editing the same kit. My context did not hold those cards.
- One grep of `contracts/recipes/*.yaml` for "halts the whole run" and "Stuck halts", to find recipe text that F6's list missed. My context held only lines 27-34 of the tapestry header and none of the other comments that say the same thing in other words.
