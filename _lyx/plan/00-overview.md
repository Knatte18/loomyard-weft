---
format: 5
approved: false
language: go
---

# Plan: intake bugs 1012

Close the board entry's four intake bugs: gate steps never name an external test package as a directory (#571, #577, #580, #587), the flaky `TestSweep` is settled without a code change (#584), a loop stop on an absent or unreadable `status.json` is diagnosable from its halt note (#581), and a plan that shapes a CLI command against the runnable-group audience rule is caught at Plan-Review (#588).

## Card Index

1 — external-test-unit-dir — map an external-test glyph unit to its package directory, and sanction the mapping in PATTERN
2 — verify-package-missing — new plan check for a Verify argument that names no directory
3 — gate-missing-package-dir — `lyx gate test` refuses a relative package argument that names no directory
4 — loop-status-unreadable — the shed loop tells an absent status file from an unreadable one and points at its log
5 — runnable-group-audience-rule — write the runnable-group audience rule where plans read it; close #584 without a change

## verify:

```bash
go build ./...
go vet -tags integration ./...
go vet -tags tmux ./...
go vet -tags llm ./...
go test ./...
go test -tags integration ./...
```
