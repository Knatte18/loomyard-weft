## Goal

```markdown
Source: `docs/research/survey/pocock-skills-2026-11-19/report.md` (survey of Matt Pocock's skills repository, 2026-11-19), candidate 2, which holds the design.

## Idea

For an entry labelled `bug`, a gate step runs the tests the diff adds or changes (named by `testreach`) once on the merge-base tree and fails when every one passes there.
The fix commit carries a `Root-Cause:` trailer that Code-Review checks like `Loosened-Check:`.
A bug with no seam for a test is a stop the record names, never a silent pass.
```

## Scope

## Decisions

## Constraints

## Auto-mode assumptions

## Open risks

## Acceptance criteria
