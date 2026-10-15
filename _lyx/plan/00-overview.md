---
format: 5
approved: false
language: go
---

# Plan: plan-test-file-path

A plan card never lists a plain `_test.go` path under `**Edit:**`: tests a card changes are member glyphs there, tests it adds are `**Create:**` `plan:` handles on the package, and the plan gates refuse a test-file path under `**Edit:**` with that way forward.
Card 1 adds the `edit-test-file` check to `internal/planparser`, wires it into the plan index's format and rework validation, and lands its spec row and docs in the same commit.
Card 2 makes the `caller-uncovered` and `redundant-file-target` ways forward stop offering a `_test.go` file, which the new check refuses.
Card 3 rewrites the plan writer's stencil to match, adds the rule that a card adding tests beside existing ones names those neighbours under `**Uses:**`, and drops the Plan-Review rubric sentence the gate now enforces.

## Card Index

1 — edit-test-file-check — Add the edit-test-file plan check and append it at the format and rework plan gates
2 — test-file-ways-forward — Gate ways forward offer only the member glyph for a _test.go file
3 — plan-stencil-test-rules — Drop the test-file exception from the plan stencil and add the Uses rule for test neighbours

## Rename mechanic

1. Run `git mv <old> <new>` FIRST, before any other change to the moved file.
2. Then make ONLY surgical edits (package declaration, imports, identifier
   retargeting) — no unrelated rewrites.
3. A genuinely new file with no predecessor belongs in a separate `Create` card, never folded
   into the `Rename` pair.
4. Never write the relocated file from scratch and delete the original — that loses
   git history exactly as an unstructured create+delete pair would.

## verify:

go build ./...
go vet -tags integration ./...
go vet -tags tmux ./...
go vet -tags llm ./...
go test ./...
go test -tags integration ./...
