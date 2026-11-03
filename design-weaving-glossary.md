# A glossary of weaving terms in English, Norwegian and other languages, marking those lym already uses

`weaving-glossary` · Task · research, docs · running · Record-Write

## Goal (operator, 2026-11-03)

LoomYard names its parts after weaving: recipes (`tapestry`, `darn`, `sampler`), modules and programs (`shuttle`, `reed`, `heddle`, `batten`, `warp`, `weft`, `fabric`, `creel`, `pawl`, `selvage`, `bolt`, `quarry`, `burler`, `webster`, `tackler`, `dobby` and more), and board artifacts (note, and from task `shed-sub-run-block` the recipes `motif` and `arras`, with `cartoon` kept for a later Plan block).
Each new name has meant searching and thinking again; one glossary to pick from ends that.

## Deliverable

`docs/research/weaving-glossary.md`: a glossary of weaving and textile terms that can serve as names in lym.

- One row per term: the English term, the Norwegian term, other languages where a term is well known (Swedish, Danish, French, German), a one-line meaning in weaving, and what kind of lym thing it would fit (a recipe, an agent role, a module, an artifact, a state).
- Cover tools and loom parts, processes and actions (including agent nouns, the one who does it, such as weaver, warper, drafter), structures and weaves, artifacts and stages (design, sample, cartoon, motif), defects and repairs, and people and trades.
- Mark every term already taken in lym, naming what holds it: search the repository's package names under `internal/`, the recipe names in `internal/loomrecipe`, the `lym` verbs (`lym --help` and its subcommands), the agent roles and seat names in the stencils, and the board's handles and statuses (`lym board labels`, `lym board list`).
  Also mark a term that is close enough to a taken one to confuse, such as `drafter` beside the task status `draft`, and say what it collides with.
- Sources: name the dictionaries and weaving references used; Norwegian terms from Norwegian weaving sources where possible, not machine translation.
- No recommendations of names for any open question; the glossary only lists and marks.

Keep the file one table per group, with semantic line breaks outside the tables.
