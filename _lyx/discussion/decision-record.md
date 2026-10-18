## Goal

Add a dev tool that reads one plan directory and shows its card dependency graph, so an operator can see which cards a plan could run in parallel and later judge a DAG batchifier on real plans.

## Scope

- A new `tools/plandag/` Go program, laid out like `tools/tokencount/`, with `plandag.sh` and `plandag.cmd` launchers.
- Input: one plan directory, such as a pair's `_lyx/plan`, parsed only through `internal/planparser`.
- Output: a markdown report holding the card by glyph/file matrix, and the card graph as Graphviz DOT; an SVG rendered with `dot -Tsvg` when `dot` is on `PATH`, and a line saying it was not rendered when it is not.
- Out of scope: batch colouring, any change outside `tools/plandag/`, and any new export in `internal/planparser`.

## Decisions

### Decision: one column per canonical ref

- **Decision:** A column is one ref any card names under a type label or `Uses:`. A ref naming a member, a glyph or a `plan:` handle, is keyed by `planparser.RefMember`'s canonical glyph string, so a `Create` card's handle and a later card's glyph of the same symbol share a column; any other ref, a file or directory path or a self glyph, is keyed by its parsed string.
- **Rationale:** reuses the parser's own canonicalisation instead of a second ref grammar, so the tool adds no plan parsing.

### Decision: a cell is change or use

- **Decision:** A card's cell in a column is `C` when the ref is one of its targets (`Card.Targets`, every type label's refs), `U` when it is only in its `Uses:`, and empty otherwise; a ref a card both targets and uses is `C`.
- **Rationale:** the edge rule needs only whether a card writes or reads a column.

### Decision: edge rule

- **Decision:** Card `i` gets an edge to a later card `j` when some column of `i` overlaps some column of `j` and at least one of the two cells is `C`. Two columns overlap when they are equal or both parse as glyphs of one unit where one names a member of the other (`pkg#T` and `pkg#T.M`), the symbol-level rule of `docs/research/parallel-batches-2026-10-17/`. Edges always run from the earlier card to the later one.
- **Rationale:** two reads never conflict, and plan order is the order webster runs cards, so edges in plan order keep the graph acyclic. An earlier read of a ref a later card changes also gets an edge, which is more conservative than the study's rule and never claims a parallelism the plan lacks.

### Decision: DOT shows the transitive reduction

- **Decision:** The DOT drops an edge already implied by a longer path. An edge whose every overlap involves a `U` cell is drawn dashed, one with a change on both sides solid. Each node is labelled `N-slug`.
- **Rationale:** reachability, and so independence, is unchanged, and a dense plan stays readable.

### Decision: report sections

- **Decision:** The markdown report holds the plan path, the matrix table (one row per card, one column per ref), a direct-dependency list per card, and a levels table: each card's depth is the longest chain ending at it, and cards of one depth have no path between them. It names the critical path length and the largest level's width.
- **Rationale:** the levels table states what could run in parallel without opening the SVG.

### Decision: invocation

- **Decision:** `go run ./tools/plandag [-out dir] <plan-dir>` writes `plandag.md`, `plandag.dot` and, with `dot` on `PATH`, `plandag.svg` into `-out`, default `.scratch/plandag`, and prints each path it wrote. The launchers make the plan directory absolute, then run from the repository root like the other `tools/` launchers.
- **Rationale:** fixed file names overwritten per run, as `tools/tokencount/` does.

## Constraints

- `PATTERN-sole-parsers` and `PATTERN-glyph-conversion-chokepoint`: plan reading only through `planparser.ParsePlan`, glyph grammar only through `glyph.Parse`.
- `PATTERN-test-isolation`: the test package runs through `tmuxkit.Main`.
- `PATTERN-test-speed`: tests are untagged, parallel and spawn no `dot`.
- `PATTERN-comment-line-breaks`: one sentence per line in Go comments.

## Auto-mode assumptions

- A darn run holds no interview; the board entry's scope section is taken as the operator's statement.
- The plan's language is Go or absent; under `language: none` no ref parses as a glyph and columns match only by equality.

## Open risks

- A column key that is neither a glyph nor a file, such as a malformed ref, matches only by equality and can miss a real overlap.
- A wide plan gives a wide matrix table; the tool does not truncate it.

## Acceptance criteria

- Two cards changing one glyph get a solid edge from the earlier to the later card.
- A card using a glyph an earlier card changes, or a member of it, gets an edge from the earlier card.
- Two cards that only use one glyph get no edge.
- An edge implied by a longer path is absent from the DOT.
- The matrix marks `C` and `U` per card and column, and a `Create` handle shares its symbol's column.
- The levels table puts independent cards at one depth.
- With no `dot` on `PATH` the tool still writes the markdown and DOT and says the SVG was not rendered.
