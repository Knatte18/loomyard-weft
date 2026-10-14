---
format: 5
approved: false
language: go
---

# Plan: intake bugs 1012

Close four intake bugs: derived and hand-written gate steps that name an external test package's `_test` pseudo-path as a directory (#571, #577, #580, #587), a loop stop on an absent or unreadable `status.json` that its halt note cannot explain (#581), the flaky `TestSweep` that no longer reproduces (#584), and a plan that shapes a CLI command against the `cmd/lyx/clitree_test.go` rules (#588).
Card 1 fixes the derived gates at their one mapping, cards 2 and 3 refuse the hand-written routes at plan time and at run time, card 4 reworks the loop's status read, and card 5 writes the audience rule where plans read it.

## Card Index

1 — external-test-unit-dir — map an external test package's glyph unit to its package directory, and name the mapping in PATTERN
2 — verify-package-missing — new plan check refusing a verify-chain package argument that names no directory
3 — gate-missing-package-refusal — `lyx gate test` refuses a relative package argument naming no directory before any slot
4 — loop-status-stop — the loop tells an absent status from an unreadable one and names the status path and its log
5 — cli-audience-rule — state the runnable-group audience rule in PATTERN-cli-cobra, the overview and the Plan-Review rubric

## Shared Decisions

- Until this plan lands, the running `lyx` derives the gate argument `./internal/planparser_test` for cards 1 and 2 and `./internal/batcher_test` for card 1, the very bug card 1 fixes; a fork runs that gate step with `./internal/planparser` and `./internal/batcher` in their place.
- Every new test is a row of an existing table except `TestValidate_VerifyPackageMissing`, whose check has no test yet; card 2 raises `internal/planparser`'s budget row for it.

## verify:

go build ./...
go vet -tags integration ./...
go vet -tags tmux ./...
go vet -tags llm ./...
go test ./...
go test -tags integration ./...
