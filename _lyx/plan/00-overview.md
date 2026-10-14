---
format: 5
approved: false
language: go
---

# Plan: intake bugs 1012

Close the board entry's four intake bugs: gate steps that name an external test package as a directory (#571, #577, #580, #587), the flaky `TestSweep` (#584, closed without a change), a loop stop on an absent or unreadable `status.json` that its halt note cannot explain (#581), and a plan that shapes a CLI command against the `cmd/lyx/clitree_test.go` rules (#588).
Card 1 maps an external test package's glyph unit to its directory, card 2 adds the plan check for author-written Verify lines, card 3 makes `lyx gate test` refuse the same mistake at run time, card 4 splits the loop's status stop by cause, and card 5 writes the CLI-shape rule where plans read it.

## Card Index

1 — external-test-unit-dir — map an external test package's glyph unit to its package directory, and name the one sanctioned mapping in PATTERN
2 — verify-package-missing — plan check refusing a Verify package argument that names no directory
3 — gate-package-dir-refusal — `lyx gate test` refuses a relative package argument that names no directory
4 — loop-status-stop — the loop stops with `status-missing` or `status-unreadable`, names its log and leaves the status file as found
5 — cli-shape-rule — the runnable-group audience rule in PATTERN-cli-cobra, the overview and the Plan-Review rubric

## Shared Decisions

- Every new test case is a row or subtest of an existing test, except card 2's `TestValidate_VerifyPackageMissing`, whose budget row card 2 raises.
- Refusal and check texts end in a `way forward:` clause naming a step an agent may run (`PATTERN-refusal-way-forward`, `PATTERN-no-denied-recovery`), and `contracts/specs/refusal-spec.md` changes in the same card as the refusal.
- D6 changes no code; card 5's commit closes #584.

## verify:

```bash
go build ./...
go vet -tags integration ./...
go vet -tags tmux ./...
go vet -tags llm ./...
go test ./...
go test -tags integration ./...
```
