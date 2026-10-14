## Changes

- F1 (LOW), `internal/loomshed/gates.go`: the header's parenthesised function list now includes `ValidateDarnDiscussion`, so each closure has its function in closure order. Committed as `b6e0b9047`.
- F2 (NIT), `contracts/specs/refusal-spec.md`: the `validate-discussion: commit failed on a darn run` row moved from between the two circling rows to just before the decision rows, text unchanged. Committed as `1d3473c03`.

Both changes touch only comment and markdown text, so no test was added or run; `lyx loom fmt` reported no change on `gates.go`.

## Disputed

None.

## Deferred

None.
