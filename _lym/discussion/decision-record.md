## Goal

```markdown
## Decision (operator, 2026-11-22)

Output only a human reads, such as the rendered README, gets no test of its layout or wording.
The general rule goes into `scribe:testing` through the scribe hub; this entry runs as darn light once that rule has landed.

## What to do

- Delete the README layout goldens in `internal/boardengine/render_test.go` and the heading parsing in `view_test.go`; do not move them to testdata files.
- Keep the tests of behaviour other code relies on, without pinning the layout: `RenderToDisk` rewriting a file in place and its manifest cleanup, and `board-data.js` holding each entry, which `board.html` reads.
- Sweep the repository for other renderers of human-read output carrying layout goldens, and delete those too.

## What was seen

In `readme-declutter` the format change touched about 200 lines of `render.go` and about 480 lines of `render_test.go`.
On main, `render_test.go` is longer than `render.go`: each run covered its format change with a new golden, and reviewers flag a change without a test.
```

## Scope

## Decisions

### Added after Discussion (operator, 2026-11-22): No test seam: the change deletes and rewrites layout tests only, with no production defect for a test to reach

Decision: The run deletes the layout goldens of human-read board output and rewrites view_test.go's placement check as literals; it changes no production code, so red-first has no failing-on-base test to demand.

Rationale: Operator, 2026-11-22: output only a human reads gets no layout tests (scribe:testing); the task exists to delete them.

## Constraints

## Auto-mode assumptions

## Open risks

## Acceptance criteria
