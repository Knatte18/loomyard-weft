# Fixer report — webster round 1

## Changes

- F1: `contracts/stencils/burler/burler-focus-directive.md` banner now names `composeReviewPrompt` in `prompt.go`, the focus directive's call site.
- F2: `internal/burlercli/cli.go` comment now cites `burlerengine.ReviewResult` and `burlerengine.FixResult`, both present.
- F3: `contracts/stencils/seat/seat-directive-advisor.md` and `seat-directive-chair.md` banners drop the clause about Burler seat stencils, none of which remain.

Each premise was checked against the code before the edit.
`lyx gate test` over `./internal/burlercli`, `./contracts/stencils`, `./internal/seatengine` and `./internal/burlerengine` passes.

## Disputed

None.

## Deferred

None.
