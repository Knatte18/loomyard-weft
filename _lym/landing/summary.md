# Add a glossary of weaving terms for naming lym parts

Adds `docs/research/weaving-glossary.md`, one table per group of weaving and textile terms that can serve as names in lym, so a new recipe, module or role name is picked from one list instead of searched for again.

Each row gives the English and Norwegian term, Swedish, Danish, French and German terms where well known, a one-line meaning in weaving, the kind of lym thing it would fit, and whether lym holds the name.
The groups are tools and loom parts, processes and actions, agent nouns, structures and weaves, artifacts and stages, defects and repairs, and people and trades.

A term lym already holds is marked `Taken:` with one or two sentences on what holds it today, taken from the package docs, each module's help, the board's help and label descriptions, and the stencil headers.
A term that could be confused with a taken one is marked `Close:` and says what it collides with, such as the agent noun drafter beside the task status `draft`.
The names the shed-sub-run-block task reserves (`motif`, `arras`, `cartoon`) and the ones board notes reserve (`weaver`, `Tenter`) are marked taken.

The glossary recommends no name for any open question.

## Design decisions

- Norwegian terms found in Store norske leksikon ("veving") and Det Norske Akademis Ordbok ("flatvev") are written plain; every other Norwegian term carries a `†` meaning it comes from general usage and needs a check against a Norwegian weaving reference.
  Web search returned no dedicated Norwegian weaving glossary, so the unmarked set is smaller than the marked one.
- The Swedish, Danish, French and German columns were not checked entry by entry against a dictionary; the Sources section says so and names the references to check them against.
- The `Taken:` descriptions follow the parent's recorded decision (`Added after Discussion`, 2026-11-03) that each taken term say what it is in LoomYard, drawn from docs and help rather than memory.
- The search for taken names ran against the repository as it stood at this commit and is described in the file, so a reader can rerun it.

## Testing

The change adds one documentation file and no code.
`lym gate test` ran on the committed tree.
