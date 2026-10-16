---
format: 5
approved: false
language: go
---

# Plan: intake-bugs-1012

Close the board entry's four intake bugs.
Derived gate steps never name an external test package as a directory (#571, #577, #580, #587).
An author-written Verify or `lyx gate test` line naming no directory is refused before it runs.
A loop stop on an absent or unreadable `status.json` is diagnosable from its halt note (#581).
The runnable-group audience rule is written where plans are written and reviewed (#588).
The flaky `TestSweep` (#584) needs no code change.

## Card Index

1 — external-test-unit-dir — map an external-test glyph unit to its package directory in planparser's target-dir mapping, and sanction that one mapping in the glyph chokepoint
2 — verify-package-missing — new plan check refusing a relative package argument in a Verify or `## verify:` command that names no directory
3 — gate-test-package-dirs — `lyx gate test` refuses a relative package argument that names no directory, before any slot or prebuild
4 — loop-status-stop — the shed loop stops with `status-missing` or `status-unreadable`, names the status file and loop log, and leaves the file as found
5 — cli-runnable-group-rule — write the runnable-group audience rule into the cli-cobra background, the overview and the Plan-Review rubric

## Shared Decisions

- **External-test members are named by glyph on their `_test` unit.**
  The `lyx` binary that runs this plan already maps an external test package's unit to its package directory, so a member glyph like `internal/planparser_test#TestCardTargetDirs` or a `plan:` handle on that unit derives the gate argument `./internal/planparser`.
  Cards therefore name a changed test function of an external `_test` package as a member glyph under Edit, and declare a new one as a Create `plan:` handle on `<dir>_test`, as `edit-test-file` requires.
- **D6, no `internal/lyxtemp` change.**
  `lyx gate test ./internal/lyxtemp -- -race -run TestSweep -count=1000` and the same with `-count=300` and no `-race` pass on current code.
  No card touches the package, and card 5's commit closes #584 citing `78e31aa06`'s bounded retry and those two runs.

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
