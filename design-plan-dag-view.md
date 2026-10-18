# Draw a plan as a card DAG with its glyph and file matrix

`plan-dag-view` · Task · feature, loom, webster · running · PR-Align

## Seen (2026-10-17)

- The parallel-batches study (`docs/research/parallel-batches-2026-10-17/`) derived each plan's card dependency graph from the cards' targets and `Uses:` with awk, but nothing shows that graph.
- A DAG batchifier for Weaver (see `weaver`) is judged on that graph, so seeing it for a real plan helps design and check the batchifier.
- Graphviz is installed on the operator's machine (`dot`).

## Idea (operator, 2026-10-17)

A tool under `tools/` takes a plan directory and draws its cards as a DAG with arrows for dependencies.

- It builds an incidence matrix: one row per card, one column per glyph and file any card names, each cell saying whether the card changes or uses it.
- Two cards with a mark in the same column get an edge; its direction follows who changes and who uses.
- It writes the matrix as a markdown table and the graph as Graphviz DOT text, rendered with `dot -Tsvg`; no Go graph package is needed for this.
- It reads targets and `Uses:` through `internal/planparser`, so no plan parsing is duplicated.

### Orch recommendation

Later, the same tool colours each card by the batch a batchifier puts it in, so a new batchifier's clusters can be checked by eye on real plans before it runs.

## Scope (operator, 2026-10-18)

- A tool under `tools/plandag/`, the way `tools/tokencount/` is laid out, with a `.sh` and a `.cmd` wrapper.
- Input: a plan directory, such as a pair's `_lyx/plan`.
- Output: the card by glyph/file matrix as a markdown table and the card graph as Graphviz DOT; it also renders SVG with `dot -Tsvg` when `dot` is on `PATH`, and says so when it is not.
- It shows what a plan could run in parallel: cards with no path between them are independent.
- Batch colouring is out of scope until a new batchifier exists.
- It changes no code outside `tools/plandag/`, apart from what reusing `internal/planparser` needs.
