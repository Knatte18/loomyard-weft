# Parts-Write reads the PATTERN entries behind the names it fixes, and its gate checks them

`parts-write-reads-patterns` · Note · enhancement, loom, stencil, pattern

## Seen (orch, plan-split-bench, 2026-11-01)

In the `plan-split-bench` replays, Parts-Write fixed export names in `parts.md` that the repository's PATTERN entries forbid, and the part writers, who may not change `parts.md`, could not correct them:
- `internal/heddlecli#NewCommand`, where `PATTERN-cli-cobra` gives every CLI package `Command()`;
- names spelling `warp` in `heddleengine`, `fabricengine` and `cmd/lym`, which `PATTERN-fabric-vocabulary` bars and `validate-plan --part` refuses as `fabric-vocabulary-name`;
- two members that keep their signatures listed as exports, which `part-export-missing` cannot accept.

Each needed a hand edit of `parts.md` to let the parts join.
Parts-Write's prompt (`loom-template-parts.md`) carries the PATTERN directive, but nothing has it read the PATTERN entries that govern the names it fixes, and its own gate does not run the name checks the part gate runs.

## Idea (operator, 2026-11-01)

- Parts-Write reads the PATTERN entries for every package a part exports from before it names an export, as the Plan writer does in its longer session.
- Its gate runs the name checks `validate-plan --part` runs on export names, `fabric-vocabulary-name` included, and refuses an export that names an existing member whose signature the part keeps, so such a defect stops at Parts-Write, where it can still be fixed.
- The `plan-split-bench` report's findings refine this before it is promoted.
