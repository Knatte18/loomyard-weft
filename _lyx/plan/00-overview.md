---
format: 5
approved: false
language: go
---

# Plan: intake bugs 1012

Close the board entry's four intake bugs: gate steps never name an external test package as a directory (#571, #577, #580, #587), the `TestSweep` flake is settled without a code change (#584), a loop stop on an absent or unreadable `status.json` is diagnosable from its halt note (#581), and a plan that shapes a CLI command against the `cmd/lyx/clitree_test.go` rules is caught before implementation (#588).

## Card Index

1 — external-test-unit-dir — map an external-test glyph unit to its package directory in planparser, the chokepoint's one sanctioned adjustment
2 — verify-package-missing — new blocking plan check for a verify argument that names no directory
3 — gate-test-missing-dir — `lyx gate test` refuses a relative package argument that names no directory before any slot
4 — loop-status-unreadable — the loop tells an absent status file from an unreadable one, leaves it as found and names its log
5 — cli-audience-rule — write the runnable-group audience rule into PATTERN, the overview and the Plan-Review rubric; close #584

## Shared Decisions

- No change to quarry or to `internal/lyxtemp` (D6); #584 closes in card 5's commit.
- Tests are unit tier and extend existing tables; only card 2 adds a top-level test, and it raises `internal/planparser`'s budget row.

## verify:

go build ./...
go vet -tags integration ./...
go vet -tags tmux ./...
go vet -tags llm ./...
go test ./...
go test -tags integration ./...
