---
format: 5
approved: false
language: go
---

# Plan: test-writing-rules

Change the rules loom tasks write tests by, so tests follow behavior and contract rather than symbols, no new test duplicates existing coverage, and every reviewer treats a redundant test as a finding.
The PATTERN half (D1, D8's PATTERN part, D10–D12) landed in the Discussion commit (D13), so no card touches `PATTERN.md` or `pattern/`.
These cards carry the rule into the plan spec, the plan writer's stencil, both review rubrics, the Burler fix step and the Discussion stencil, and move the wording claims in `contracts/stencils/claims_test.go` onto the new phrases.

## Card Index

1 — plan-spec-card-model — the plan spec's card criterion, granularity and worked-example prose follow coverage and the public surface
2 — plan-template-coverage — the plan writer's stencil asks for coverage, not a bundled test file, and its claims follow
3 — review-rubrics-test-economy — Plan-Review granularity by public surface, and both rubrics flag redundant tests
4 — fix-and-discussion-stencils — the Burler fix adds a test only for a coverage gap, and must-cover scenarios are behaviors

## Shared Decisions

- Every rubric and stencil edit points at `PATTERN-test-economy` and restates it at most as a finding condition or a one-clause card criterion (`PATTERN-producer-pointer-rule`).
- Wording pins change as rows and claims of the existing `wordingClaims` table (D9); no card adds a `Test` function.
- D8's `refusal-spec.md` half needs no card: the file states no per-refusal test obligation.
  Its only test wording is the wiring-guard class's "no per-row test" and the wiring-guard row's "none per row; grouped", which D8 keeps because they say those rows need no reaching test at all.
- The planparser golden fixture and the worked example's card bodies stay byte-identical; card 1 edits only the prose above them.
- `scribe:testing` lives in the scribe repository and is not edited; the landing PR description carries the decision record's `## Notes for the plan writer` wording for it, so the hub can carry it there.

## verify:

go build ./... && go test ./... && go test -tags integration ./contracts/... && go vet -tags smoke ./contracts/...
