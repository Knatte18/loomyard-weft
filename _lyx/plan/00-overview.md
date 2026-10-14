---
format: 5
approved: false
language: go
---

# Plan: intake bugs — external-test gate paths, loop status stops, CLI shape rule

Close the board entry's four intake bugs: gate steps never name an external test package as a directory (#571, #577, #580, #587), the `TestSweep` flake is settled without code (#584), a loop stop on an absent or unreadable `status.json` is diagnosable from its halt note (#581), and a plan that shapes a CLI command against the `cmd/lyx/clitree_test.go` rules is caught at Plan-Review (#588).

## Card Index

1 — external-test-unit-dir — map an external-test glyph unit to its package directory in CardTargetDirs and sanction it in the glyph chokepoint
2 — verify-package-missing — new plan check refusing a Verify package argument that names no directory
3 — gate-package-refusal — lyx gate test refuses a relative package argument naming no directory before any slot
4 — loop-status-unreadable — the shed loop tells an absent status file from an unreadable one and points at its log
5 — cli-shape-rule — write the runnable-group audience rule into PATTERN, overview and the Plan-Review rubric

## Shared Decisions

- D6 lands no `internal/lyxtemp` change; card 5's commit message closes #584.
- The `_test` hint in card 2's finding and card 3's refusal names the same corrected directory: the argument with its `_test` suffix removed.

## verify:

go build ./...
go vet -tags integration ./...
go vet -tags tmux ./...
go vet -tags llm ./...
go test ./...
go test -tags integration ./...
