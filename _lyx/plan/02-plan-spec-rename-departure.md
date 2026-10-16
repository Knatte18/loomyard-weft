# Card 2 — plan-spec-rename-departure

**Prosa:**
- `contracts/specs/loom-plan-spec.md`
- `contracts/stencils/loom/loom-template-plan.md`

**Intent:**
D1.5, D1.6 and D2.1, the spec text the later cards point at.

`loom-plan-spec.md`:

- The `## Card types` table's `Rename` row's mechanical-check cell reads "the hub's structure-aware rename tool (`language.yaml` `rename_tools`) + grep verify, never text or regex replace".
- The `## Quarry integration` Rename bullet drops its per-language library list (`go/ast` + `go/types`, Roslyn's `Renamer`, `rope`/`libcst`) for the same pointer: the hub's structure-aware rename tool that `language.yaml`'s `rename_tools` names, then the grep pass for zero remaining old-name references and the updated doc comment.
  It keeps its quarry sentence.
- The `Rename` carve-out paragraph's "AST-aware rename" wording becomes "structure-aware rename".
- The canonical `## Rename mechanic` block under `## Rename and Move`, and the worked example's copy of it, separate the two pair kinds.
  The four current steps become the file pair's steps, the first reading "For a file pair (both sides file self glyphs or paths), run `git mv <old> <new>` FIRST, before any other change to the moved file."
  A fifth step reads: "For a symbol pair, run the hub's structure-aware rename tool the implementer prompt names (`language.yaml` `rename_tools`), never a text or regex replace, then grep for zero remaining old-name references and confirm the symbol's doc comment opening names the new name."
  The paragraph after the block says the section is required by `rename-mechanic-missing`, whose condition is unchanged, and that a hub with an empty `rename_tools` list cannot run a symbol pair.
- A new `## Pattern-Departure trailer` section, after `## Numbering and commit subject`, owns the trailer shape:
  a Webster fork, the recovery strand or the verify-fix fork that departs from its card because a PATTERN entry forbids the card's way records each departure as one trailer line in the departing commit's body, `Pattern-Departure: <PATTERN-entry-id> — <what differs from the card and why>`, in the commit message's last paragraph, where git reads trailers.
  A departure from an entry the task declares changed in the overview's `## PATTERN changes` cites the declared new text, never the old.
  `record-batch` and `recover-batch` read the trailers of the batch's own non-merge first-parent commits into the digest's `departures`, and the verify gate reads the verify-fix fork's; Webster-Review reads them in the commit messages of its range.

`loom-template-plan.md`: its `### `## Rename mechanic` — reproduce verbatim` block carries the same five-step text as the spec's canonical block, byte for byte.

Both are static prose; no test reads them.
