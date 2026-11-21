## Goal

```markdown
## Seen

2026-11-19: in `raddle-slug-length-check`, the red-first gate (landed with `bug-red-first-gate`) could not judge the diff because it also changed `contracts/specs/raddle-proposal-spec.md`.
The base overlay cannot put that file back, so the run asked the parent for a "No test seam" waiver although the bug has a Go seam: `TestRaddleBoardContext` and the new `TestCheck` rows fail on the merge base.

## Idea

The overlay restores only the Go files the red-first check needs on the merge base and leaves non-Go files that no test reads at HEAD,
or it puts back any tracked file, so a doc or spec update beside a fix no longer waives the gate.
```

## Scope

## Decisions

### Added after Discussion (parent, 2026-11-21): No test seam for the installed red-first gate: it predates this fix

Decision: Waive the red-first gate for this run and keep the stencil edit to contracts/stencils/darn/darn-template-write.md.

Rationale: The installed lym holds the bug this task fixes, so it refuses the stencil edit as a non-Go file. The fix's own tests (TestBaseOverlay_ReplacesTheFilesOfTheModuleTheBasesBuildReads and the new TestGateRedFirst rows) fail on the merge base.

## Constraints

## Auto-mode assumptions

## Open risks

## Acceptance criteria
