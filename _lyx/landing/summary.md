# plandag draws a plan's cards as a dependency graph with their ref matrix

`tools/plandag/` is a new dev tool that reads one plan directory and shows which of its cards depend on which, and so which could run in parallel.
It is meant for checking a DAG batchifier for webster against real plans.

## What it does

`tools/plandag/plandag.sh <plan-dir>` (or `.cmd`, or `go run ./tools/plandag [-out dir] <plan-dir>`) parses the plan through `planparser.ParsePlan` and writes into `.scratch/plandag/`:

- `plandag.md`: a summary line (cards, direct dependencies, critical path, widest level), a levels table, each card's direct dependencies, and the card by ref matrix.
- `plandag.dot`: the dependency graph as Graphviz DOT, one node per card labelled `N-slug` with the Card Index summary as tooltip.
- `plandag.svg`: rendered with `dot -Tsvg` when `dot` is on `PATH`; without it the tool says no SVG was rendered and prints the command that renders it.

## How dependencies are derived

- The matrix has one row per card and one column per ref any card names.
  A cell is `C` when the card names the ref under a type label (`Card.Targets`) and `U` when it names it only under `Uses:`.
- A member ref, a glyph or `plan:` handle, is keyed by `planparser.RefMember`'s canonical glyph, so a `Create` card's handle and a later card's glyph of the same symbol share a column.
  Any other ref is keyed by its own string.
- A later card depends on an earlier one when a column of each overlaps and at least one cell is `C`; two uses never make an edge.
  Columns overlap when equal, or when both are member glyphs of one unit and one names a member of the other (`pkg#T` and `pkg#T.M`), parsed through `glyph.Parse`.
  This is the symbol level of `docs/research/parallel-batches-2026-10-17/`.
- Edges run from the earlier card to the later one, so the graph is acyclic.
- A card's level is the number of cards on the longest chain ending at it; cards of one level have no path between them.

## Design decisions

- An earlier card that uses a ref a later card changes gets an edge too.
  The study counted only the reverse direction; this is more conservative, so the graph never shows two cards as parallel while one rewrites what the other reads.
- The DOT holds the transitive reduction: an edge implied by a longer path is dropped, which keeps reachability, and so independence, unchanged.
  The markdown's dependencies table still lists every direct edge.
- An edge where both cards change an overlapping ref is solid; one where a side only uses it is dashed.
- `run` takes the resolved `dot` binary as a parameter, so tests exercise the no-`dot` path without touching `PATH` or spawning `dot`.
- The plan directory is a required argument, with no default to the current worktree's plan, which would need cwd resolution through `internal/lyxcwd`.
- No `internal/planparser` change was needed.

## Limits

- A file self glyph (`pkg/f.go#`) and a member glyph of the same package (`pkg#X`) do not overlap, because the tool cannot tell which file declares `X`.
- Under `language: none`, or for a malformed ref, columns match only by equality.

## Tests

`tools/plandag/main_test.go` drives `run` on a fixture plan and compares the written report, DOT and stdout exactly.
The fixture covers change-change and member overlap, a handle sharing its glyph's column, use-only pairs with no edge, a dropped implied edge, an earlier use before a later change, and the no-`dot` message.
A second test pins that an unparseable plan fails with no output directory written.
