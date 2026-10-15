---
format: 5
approved: false
language: go
---

# Plan: lighter test rules for stencil prose

A change to a stencil's static prose should cost no test change.
This plan deletes `contracts/stencils/claims_test.go`, moves the rows that pin Go-declared values, the twill host-neutral ban and the Go-filled marker positions into kept tests of `contracts/stencils`, prunes every other assertion on stencil or rendered-prompt text by the decision record's D1 rule, and changes the plan spec, the plan stencil, `PATTERN-test-economy` and the review rubrics so review, not a substring test, guards prompt wording.

## Card Index

1 — replace-stencil-claims — delete claims_test.go and land its D1 keeps as the Go-value table, the twill host-neutral scan, the marker-position table and new marker-set rows
2 — webster-template-tests — prune webster's rendered-prompt test to Go-filled, Go-chosen and Go-named text
3 — burler-prompt-tests — prune Burler's prompt tests to Go-built and Go-chosen text
4 — treadle-template-tests — pin treadle's judge templates to the verdict words and keys Go parses
5 — shed-prompt-tests — prune the tackler, driver and Bouncer judge prompt tests and drop the driver stencil's shape test
6 — loom-prompt-tests — prune the loom, alignment and darn prompt tests
7 — directive-tests — prune the directive and orch pointer tests
8 — coverage-rule-text — exempt stencil prose from the coverage rule in the spec, plan stencil and PATTERN, and add the rubric guards

## Shared Decisions

- D1 sorts each assertion, not each file: an assertion on stencil or rendered-prompt text stays when it checks a marker set, a required-marker error, a Fill round-trip or a Go-filled marker's position; a value Go filled, or a fragment Go chose, anchored on the shortest phrase that tells the branches apart; a Go-declared value read from its Go definition; or a rule across a stencil family. Every other assertion on stencil prose goes.
- Text built in Go source (refusals, Go-built prompt blocks such as Burler's cluster rules or the Bouncer's decision rule, Go-built commands) is out of scope and keeps its assertions.
- A test left with no assertion is deleted; a helper left unused is deleted.
- Prose a recipe carries, such as the Darn-Burler-Review fasit instructions in `contracts/recipes/darn-recipe.yaml` that `internal/loomrecipe`'s `TestNewDarn_RowsMatchTheDarnTable` asserts, is not stencil text, so its assertions stay: the acceptance criterion and D1 name stencil prose, and the record lists no recipe.
- `cmd/lyx/testdata/test-budget.yaml` is not lowered (D5).

## verify:

```bash
go build ./...
go vet -tags integration ./...
go vet -tags tmux ./...
go vet -tags llm ./...
go test ./...
go test -tags integration ./...
```
