---
format: 5
approved: false
language: go
---

# Plan: batcher-v2

Make the cost batchifier's start-context model measured instead of guessed: a fork starts at Merriam's base plus a growth per batch before it, the batchifier is told where its cards sit in the run, and `tools/tokencount -calibrate` fits those coefficients and reports each profile's cost and quality from finished runs through one repository flag.
Alongside, the plan spec and stencils state that a plan runs in card order, Plan-Gate refuses a card command that reaches the fabric repo through a check sharing the implementer audit's rule, and `accept-audit --batch` accepts a committed batch whose fabric references only read.

## Card Index

1 — batcher-start-model — the start context becomes master_base plus batch_growth per batch position, through estimator, dynamic program, breakdown and template
2 — handover-position — the batchifier is told how many batches run before its cards; webster's first init, validate and rebaseline pass it
3 — tokencount-one-repo-flag — one flag names the repository for tags, plan commits and webster records; the history flag goes
4 — tokencount-fork-start — each fork records its start context and its Merriam session
5 — calibrate-per-fork — calibration fits the start coefficients by least squares and estimates every fork at its position
6 — profile-cost-quality — calibration adds a per-batch cost table and a per-profile cost and quality summary
7 — card-order-rule — the plan spec, template and review rubric state that a plan runs strictly in card order
8 — location-free-ref-rules — the audit's location-free fabric-reference patterns become a rule callable on its own
9 — plan-fabric-reference-check — planparser's card-fabric-reference check over Verify values, verify section, fenced blocks and inline spans
10 — gate-fabric-reference — both plan gates and validate-plan run the check; the spec row and the committed-data steer land
11 — accept-committed-readonly — accept-audit --batch accepts a committed batch whose fabric-reference commands are read-only

## Shared Decisions

No card command reads the fabric repo or another worktree's files, or runs `tools/tokencount` against them: every fixture is built in a temp directory or committed under the tree, and the template's coefficients come from the decision record's measurement.
Test files never name the fabric side in an identifier, string or comment (`PATTERN-fabric-vocabulary`): they take sibling paths from `fabricengine.WeftWorktree`, spellings from `lyx fabric …` or the three exempt spellings, and tokencount's flag name from its non-test constant.

## verify:

go build ./...
go vet -tags integration ./...
go vet -tags tmux ./...
go vet -tags llm ./...
go test ./...
go test -tags integration ./...
