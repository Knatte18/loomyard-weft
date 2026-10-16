---
format: 5
approved: false
language: go
---

# Plan: orch-intake-fork

The orch learns whether the GitHub inbox holds anything from issue numbers and titles alone, and a one-shot fork does the intake, so no issue body enters the orch's context.
`lyx board intake list` turns compact by default with the full view behind `--full`, `intake import` prints a short result, and the orch role stencil and the `ly:board` skill state the new check, the fork's bounds and the new output.
`internal/boardcli/doc.go` states each output change in the card that makes it.

## Card Index

1 — intake-list-compact — `intake list` prints numbers and titles, `--full` prints today's view
2 — intake-import-short — `intake import` prints slug, issue, labels and dropped instead of the entry
3 — orch-intake-fork — the orch stencil checks the inbox and forks intake, and `ly:board` matches the new output

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
go vet -tags integration ./internal/boardcli/... ./contracts/...
go vet -tags tmux ./internal/boardcli/... ./contracts/...
go vet -tags llm ./internal/boardcli/... ./contracts/...
go test ./internal/boardcli/... ./contracts/...
go test -tags integration ./internal/boardcli/... ./contracts/...
```
