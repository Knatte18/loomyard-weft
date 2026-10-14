---
format: 5
approved: false
language: go
---

# Plan: intake bugs 1012

Close the board entry's four intake bugs: gate steps never name an external test package as a directory (#571, #577, #580, #587), a loop stop on an absent or unreadable `status.json` is diagnosable from its halt note (#581), the flaky `TestSweep` is settled without a change (#584), and a plan that shapes a CLI command against the `cmd/lyx/clitree_test.go` rules is caught at review (#588).
Card 1 fixes the derived gates at their source, cards 2 and 3 catch the hand-written route at plan time and at run time, card 4 reworks the loop's status-read stop, and card 5 writes the audience rule where plan writers and reviewers read it.

## Card Index

1 — external-test-unit-mapping — map an external-test glyph unit to its package directory in planparser, so card and batch gates and the batcher's estimate name the package
2 — verify-package-missing — add a blocking plan check for a relative Verify package argument that names no existing or created directory
3 — gate-missing-package-refusal — refuse a lyx gate test relative package argument that names no directory, before any slot or prebuild
4 — loop-status-read-stop — stop the shed loop with status-missing or status-unreadable, naming the status path and loop log, and leave the file as found
5 — runnable-group-audience-rule — state the runnable-group audience rule in PATTERN-cli-cobra and point overview.md and the Plan-Review rubric at it

## Shared Decisions

- Quarry is untouched: its glyph grammar, `Glyph.UnitPath` and the external-test pseudo-unit convention stay as they are; `planparser`'s `goPackageUnit` is the one sanctioned adjustment.
- `internal/lyxtemp` does not change (D6); card 5's commit body closes #584.
- Each refusal and stop names a step an agent may run, and its `refusal-spec.md` or `loom-plan-spec.md` row changes in the same commit.

## verify:

go build ./...
go vet -tags integration ./...
go vet -tags tmux ./...
go vet -tags llm ./...
go test ./...
go test -tags integration ./...
