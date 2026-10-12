---
format: 5
approved: false
language: go
---

# Plan: plan-gate-fixes

Stop the plan gate's `delete-before-reference` check from flooding plans with false blocking findings by matching a deleted member through the typed reference computation `caller-uncovered` already runs, scan-only and severity-aware at dispatch and in Webster (issue #562).
Then let the Plan-Review reviewer check its proposed fixes against the plan gate through a new in-memory `validate-plan --overlay` mode, and name the enclosing function for a comment-tied glyph (issue #563).

## Card Index

1 — shared-reference-delete-order — delete-before-reference reads caller-uncovered's reference computation, typed at the plan gates and scan-only at dispatch, over Go code only
2 — webster-blocking-delete-references — Webster's batch recording and recover-batch count only blocking delete-before-reference findings
3 — plan-overlay-reader — planparser parses and rewrites a plan through an in-memory overlay of existing plan files
4 — overlay-through-checks — canonicalization, its re-parse and the done-card comparison read through the plan's own source
5 — validate-plan-overlay-flag — validate-plan --overlay reads proposed plan contents from stdin and checks them in memory
6 — plan-review-rubric-overlay — the Plan-Review rubric validates proposed fixes with --overlay and names a comment's enclosing function

## Shared Decisions

- One reference computation, `memberReferences`, serves `caller-uncovered` and `delete-before-reference`; a plan-gate pass runs it, and so the type load, once.
- An overlaid plan carries its overlay on `planparser.Plan`, and every plan-file read or write in the check path goes through `Plan.ReadFile`, `Plan.RewriteRefs` and `Plan.Reparse`.

## verify:

go build ./...
go vet -tags integration ./...
go vet -tags tmux ./...
go vet -tags llm ./...
go test ./...
go test -tags integration ./...
