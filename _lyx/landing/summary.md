# Plan gate: typed delete-before-reference, validate-plan --overlay

The plan gate's `delete-before-reference` check matched a deleted method by its bare name, so one deleted `Run` or `Call` flagged every later `.Run(` or `.Call(` and every prose mention.
One plan drew about 220 false blocking findings from a handful of deletes, and the only admissible shape was to move every deletion into the last card.
This change resolves references the way `caller-uncovered` does.
It also lets the Plan-Review reviewer check its proposed fixes against the plan gate before writing them.

Closes #562
Closes #563

## What changes

- **`internal/planglyph`:** for a deleted member glyph, `delete-before-reference` takes the references from `caller-uncovered`'s typed load and scan, restricted to later cards' Edit regions.
  Each plan-gate pass runs one type load shared by both checks.
  A reference inside a later Edit region belongs to `delete-before-reference`, and every other uncovered one stays with `caller-uncovered`, so no line is reported twice.
- **Severity follows resolution:**
  - A typed reference, or a scan-resolved package-level one, is blocking.
  - A method reference matched by name alone, where no type information answered, is one informational finding per deleted method.
  - Only Go code is searched, so comments, strings and non-Go Edit targets never match.
  - Go files under `testdata`, `vendor`, or a `.`- or `_`-prefixed directory are skipped, as `caller-uncovered` skips them.
  - A package self glyph Delete keeps its quoted import-path match, now limited to `.go` Edit regions.
- **`internal/websterengine`:** batch recording and `recover-batch`'s `ErrRecoveryDeleteReferenced` refusal count only blocking `delete-before-reference` findings.
  `ValidateDispatch` and `LaterDeleteReferences` run the token scan with no type load, so a name-only method match there is informational.
- **`lyx loom validate-plan --overlay`:**
  - It reads a JSON object from stdin mapping plan file names to proposed full contents and validates the plan with them in memory.
  - Nothing is written to disk, including handle canonicalization, which now substitutes in memory.
  - An overlaid done card is reported as edited, as the same edit on disk would be.
  - It refuses, each with a way forward and a row in `refusal-spec.md`, for bad stdin, a bad name, and a combination with `--rework` or `--require-approved`.
- **`internal/planparser`:** a plan can now be parsed and rewritten through an in-memory overlay source.
- **Plan-Review rubric (`contracts/stencils/loom/loom-rubric-plan-review.md`):** before proposing a fix that changes targets or a verify command, the reviewer runs `validate-plan --overlay`.
  It compares against the `{}` baseline and withholds any fix that adds a blocking finding.
  A glyph tied to a comment must name the enclosing function, confirmed with `lyx quarry resolve`, or the file's self glyph when no member span holds the line.
- Docs: `internal/planglyph/doc.go`, `internal/loomcli/doc.go`, `internal/planparser/doc.go`, `internal/websterengine/doc.go`, `contracts/specs/loom-plan-spec.md`, `contracts/specs/refusal-spec.md` and `pattern/PATTERN-gate-self-check-parity.md`.

## Design decisions to review

- **Package-level members move to the shared computation too.**
  The issue names methods, but one computation serves both kinds, so their matching also moves from a bare-word regex to the import-path scan or the typed load.
- **Name-only method matches are informational.**
  Where no type information exists (failed load, parse error, nested module, untyped identifier, dispatch and Webster paths), a real method call surfaces as informational and the later card's verify build catches it.
- **Dispatch and Webster stay scan-only.**
  They re-validate against a tree the run has changed and carry no loader, and a type load in Webster's failure path could wait on a gate slot it already holds.
- **The overlay replaces existing files only.**
  Inserting, removing or renumbering cards still goes through `insert-card` and `remove-card`.
  It never changes the tree the glyphs resolve against, so it validates target lists, verify commands and card text, not proposed code changes.
- **`README.md` and `internal/lyxcwd/enforcement_test.go`** changed so the fabric-vocabulary tree scan accepts the records repository's name in `CLAUDE.md` and `README.md`.

## To check by hand

- A reviewer may skip the overlay run; the rubric cannot force it, and the fixer's own gate remains the backstop.
- The overlay run adds Plan-Review time, which shifts the cost baseline `twill-plan-followups` measures Plan-Review rounds against.
- The full plan verify was not run after the vocabulary-scan fix commit `a2618efa4`.
  Only `lyx gate test --tags integration ./internal/lyxcwd` was rerun, and it passes.
- Two gate steps, `./internal/websterengine_test` and `./internal/planparser_test`, name external test packages and fail with `directory not found`.
  The packages pass when run without those steps; the step names may need correcting.
- `internal/planparser/parse.go` changed beyond its planned edits: the overview-reading step moved out of `ParsePlanFrom` into a helper so the overlay constructor reads the Card Index the same way.
