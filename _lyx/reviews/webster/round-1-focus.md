---
round: 1
focus:
  - "Cards 1, 2 and 4: check that open maps are carried whole through every path (Load, LoadOrTemplate, Set, Reconcile, FillMissing, MissingKeys, SetValues), and that no existing caller or `PATTERN-config-strictness` pinned call site changed signature."
  - "Card 3: check that `board.yaml` accepts both the list shape and the ordered-map shape for `types` and `labels`, that file order is kept, and that `Vocabulary` and `Outputs` still return the same `[]string` values."
  - "Cards 5 and 6: check that `lyx config --set labels.<name>=...` adds or rewrites exactly one label without dropping others, and that `lyx board labels` prints both maps in file order."
  - "Cards 7 to 9: check `get --body` prints the body verbatim, `merge --body-file` is wired and validated, and a merge carries the removed entries' issues without duplicates or loss."
  - "Cards 10 to 14: check that each card's mechanical check ran against every target: the rename of `maxSlugLength` leaves no stale reference, every slug-taking board verb's help reads the constant, and `cliTreeFindings` has no remaining caller."
  - "Docs and comments: check new `doc.go` files (`yamlengine`, `configengine`, `configcli`, `boardcli`) and `SKILL.md` edits follow semantic line breaks and the repo's comment conventions, and that `PATTERN.md` and `docs/overview.md` changed where behaviour or the module table did."
---

Seeded from the plan overview's card index and shared decisions.
Entries name where to look; none states a verdict.
