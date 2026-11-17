# Add a code-review lens that flags unrequested scope

Code review now checks the diff against the decisions that asked for it.
A new `code-scope` lens maps each change in the diff to a decision and reports two gaps:
a change that traces to no decision, and a decision that no change builds.
Both are findings of class `scope`.
Until now nothing checked a diff for work no decision asked for, although decisions carry IDs.
The idea comes from Spec Kit's `/converge`, which classifies gaps as missing, partial, contradicting or unrequested.

## What changes

- `internal/lenses/template.yaml` defines the `code-scope` lens and adds it to the `review-code` fan.
- `internal/lenses/sets/csharp.yaml` adds it to that set's `review-code` fan.
- `docs/dotnet-hub.md` adds it to the `review-code` fan example.
- `internal/lenses/config_test.go` updates the seeded-template expectations: the lens total, the `review-code` entry count and its member list, and the remove-all-members case.

## Design decisions

- The lens builds its decision list from whichever answer key the run has.
  With a plan, the decisions are the `### D<n> — <title>` headings of the record's `## Decisions` section, and each change is mapped through the plan card that carries its decision.
  With a decision record alone, they are the goal points of the board entry under its `## Goal`, never the record's other sections.
  In both cases the record's `Added after Discussion` entries count as decisions.
- A change that traces to no decision is not flagged when a traced change forces it (a caller, a test, a doc or a rule document it makes stale) or when a `Pattern-Departure:` trailer explains it.
- The lens defers to the rubric's "Scope against the answer key" for how a scope finding is judged.

## To check by hand

- `code-scope-writer-claimed-decision-source` (class design, MEDIUM): the lens text in `internal/lenses/template.yaml` takes its decision list from the record and board entry.
  Check that it does not rely on what the change's author claims as the decision source.
- `dotnet-hub-review-code-fan-omits-code-scope` (class consistency, MEDIUM): the `review-code` fan example in `docs/dotnet-hub.md` now lists `code-scope`.
  Check that every other place that enumerates the fan matches.
