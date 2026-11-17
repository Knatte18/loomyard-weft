# A review lens that flags changes no decision asked for

`unrequested-scope-lens` · Task · feature, lenses, burler · running · Drift-Expect

## Seen (orch, 2026-11-17)

The decision record's decisions carry IDs (`internal/discussionparser/decisionids.go`), but no review checks a diff for work no decision asked for.
Source: the survey F17 (`orchestrator-survey-ideas`), from Spec Kit's `/converge`, which classifies gaps as missing, partial, contradicting or unrequested.

## Idea

- A lens in code review that maps each change in the diff to a decision ID and flags a change that traces to none as unrequested scope.
- It also flags a decision no change builds.
