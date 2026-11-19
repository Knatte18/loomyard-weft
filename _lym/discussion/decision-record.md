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

### Added after Discussion (parent, 2026-11-19): Red-first gate on darn runs only

Decision: Accepted: the red-first gate runs in Darn-Write only; tapestry and sampler runs of a bug entry are not checked in this run.

Rationale: Bug fixes run as darn by default on this hub; tapestry gates per Webster batch, a different seam with its own design questions, so extending the gate there is a separate change if bug runs on tapestry show the need.

## Constraints

## Auto-mode assumptions

## Open risks

## Acceptance criteria
