```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the review proposals the fixer carried out as proposed (F1, F3–F9, F11, and the F10 ruling) by building cases where they go wrong.

**MEDIUM — F7 (card 2 `Open`: an absent `sharedDir` reads as a directory with no maps)**
- **What breaks:** the proposal answers card 11's triage case, but it also changes the reach step (card 5) and the reach turn (card 9).
- **Concrete case:** a hub whose `lym gate test --map` never ran has no shared maps.
  - `Open` now succeeds with no maps, so card 5's step writes a matching `reach.json` that lists every changed function as reached by no mapped test.
  - That report counts as judged, because nothing in card 5 treats "no map read" as judging nothing.
  - On a passing verify, card 9's `WithReachTurn` sees unreached functions no turn listed yet. It returns a `WriterOnly` failure telling the writer to "add the missing tests and commit them" for every changed function.
  - The cause is that no maps exist, not that tests are missing, so the turn costs a writer turn and a gate attempt on a false note.
- **Evidence:**
  - Card 2's changed text: "An absent `sharedDir` or `worktreeDir` reads as a directory with no maps and refuses nothing … a hub whose `--map` never ran are common."
  - Card 5's judged-nothing cases list only a red tier, outside a hub, a non-Go scoper and no parent branch.
  - Card 9: "With a pending report holding a function whose id and hash the listed file does not hold … returns one failing result with `WriterOnly` set".
  - Card 7 renders the same report into the review as current evidence.
- **Needed:** a sibling rule in card 5. When `Maps()` is empty, the report judges nothing, and its note says no coverage map exists and to build them with `lym gate test --map` from the prime.

**LOW — F6 (card 13: past `mutation_budget_min`, "every mutant left is reported unjudged")**
- **The gap:** the new budget row expects "the planned mutants of a package whose anchor map matches are reported unjudged" while the budget is "spent at the hold".
- **Why it matters:** mutants exist only after the coverage binary is built and the block table read, and the card puts that work inside the slot bounded "by `mutation_budget_min` from the hold". The new sentence stops only "further mutant build or run".
- **The two readings:**
  - If the bound also stops the coverage build and planning, the row has no planned mutants to report.
  - If it doesn't, a package's coverage build runs past the budget.
- **Needed:** the card should say whether the coverage build and planning run past the budget, so the row's expectation follows from the text.

**No finding on the rest:**
- **F1:** the refresh holds the map run's slot ask, and `lym gate mutate` is a one-slot holder after it. This matches the gateslot doc's holder model.
- **F3:** a grep found no other test replacing a template line that carries `reports`.
- **F5:** with `limit` 2, the walk returns 3 commits without meeting `--since`, so the refusal is reached.
- **F8:** a record that keeps failing passes on every arrival and never loops.
- **F9:** a review only reads a task worktree's report, so the new prime reason is harmless.
- **F10:** the scenario row asserts "not in the coverage maps", which does not depend on the change set.
- **F11:** the `site{...}` literals in `internal/testcover/mutant_test.go` use the test package's own `site` type, so adding a field to the production `site` breaks no test.

Tool calls:
1. Grepped `internal/testcover` for `site{` literals and `[]site` comparisons, to see whether a new `site` field breaks positional literals. My context had `mutant.go`'s struct but not its tests.
