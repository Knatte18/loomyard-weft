# Fixer report — plan, round 1

## Changes

- F1 (fixed as proposed): in `_lyx/plan/01-master-read-set.md`, the opening-paragraph bullet now says "you read the plan's overview once, up front, plus the rules a constraints section below names, when that section is present", so the rules read is conditioned on the constraints section.
  The new wording holds no `pattern/` string, so the card's count check stays equal.
- F2 (fixed as proposed): in `_lyx/plan/00-overview.md`'s verify section, `./tools/sandbox/...` is appended to the three `go vet` lines and both `go test` lines; `go build ./...` is unchanged.

`lyx loom validate-plan` reports ok.

## Disputed

None.

## Deferred

None.
