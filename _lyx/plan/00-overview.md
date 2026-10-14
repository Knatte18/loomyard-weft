---
format: 5
approved: false
language: go
---

# Plan: intake bugs — external-test gate paths, verify package check, loop status faults, CLI audience rule

Close the board entry's four intake bugs.
Webster's derived gates and the batcher stop naming an external test package as a directory (#571, #577, #580, #587), and the plan check and the gate refusal catch the same mistake in author-written lines.
The shed loop tells an absent status file from an unreadable one and points its stop at its own log (#581).
The runnable-group audience rule is written where plans read it (#588).
The flaky `TestSweep` (#584) closes with no code change.

## Card Index

1 — external-test-unit-dir — map an external-test glyph unit to its package directory and sanction it in the glyph chokepoint
2 — verify-package-missing — new plan check refusing a Verify package argument that names no directory
3 — gate-missing-package-dir — `lyx gate test` refuses a relative package argument that names no directory
4 — loop-status-fault — the shed loop tells an absent status file from an unreadable one and names its log
5 — runnable-group-audience-rule — the runnable-group audience rule in PATTERN, the overview and the Plan-Review rubric

## Shared Decisions

- D6: no change to `internal/lyxtemp`; card 5's commit message closes #584.
- Card 2 uses card 1's `externalTestUnitDir` for its replacement hint, so it runs after card 1.

## Rename mechanic

1. Run `git mv <old> <new>` FIRST, before any other change to the moved file.
2. Then make ONLY surgical edits (package declaration, imports, identifier
   retargeting) — no unrelated rewrites.
3. A genuinely new file with no predecessor belongs in a separate `Create` card, never folded
   into the `Rename` pair.
4. Never write the relocated file from scratch and delete the original — that loses
   git history exactly as an unstructured create+delete pair would.

## verify:

```bash
go build ./...
go vet -tags integration ./...
go vet -tags tmux ./...
go vet -tags llm ./...
go test ./...
go test -tags integration ./...
```
