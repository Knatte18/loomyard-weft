---
format: 5
approved: false
language: go
---

# Plan: Orch role stencil fixes from the stencil intent audit

Bring the orch role stencil in line with the operator's decisions of 2026-10-16: the arbiter is the orch's filter for a stuck review and every other review escalation is a fault, comparing related runs is the alignment agent's job, a small PR-Gate defect is fixed through the reject, and the stencil carries nothing specific to one repository.
Each repository gets one orch-only rules file, `ORCHESTRATOR.md` at the prime worktree's root, which every role-file render fills into the role's last section, and this repository's orch-only rules move into it from `CLAUDE.md`.

## Card Index

1 — role-file-repository-rules — read `ORCHESTRATOR.md` from a told path and fill a second role-stencil marker with it at every role-file render
2 — role-stencil-rewrites — rewrite the role stencil's escalation, stop, PR-Gate, landing, finding and notes prose to D1–D5
3 — escalation-brief-operator-decisions — the escalation brief settles only on a recorded operator decision and decides nothing at cause `budget`
4 — orchestrator-md — create `ORCHESTRATOR.md` with the moved orch-only rules, cut them from `CLAUDE.md` with a pointer line, and add the #673 board note

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
go vet -tags integration ./internal/orchengine/... ./internal/orchcli/... ./contracts/stencils/... ./internal/shedadapters/... ./internal/lyxcwd/... ./cmd/lyx/...
go vet -tags tmux ./internal/orchengine/... ./internal/orchcli/... ./contracts/stencils/... ./internal/shedadapters/... ./internal/lyxcwd/... ./cmd/lyx/...
go vet -tags llm ./internal/orchengine/... ./internal/orchcli/... ./contracts/stencils/... ./internal/shedadapters/... ./internal/lyxcwd/... ./cmd/lyx/...
go test ./internal/orchengine/... ./internal/orchcli/... ./contracts/stencils/... ./internal/shedadapters/... ./internal/lyxcwd/... ./cmd/lyx/...
go test -tags integration ./internal/orchengine/... ./internal/orchcli/... ./contracts/stencils/... ./internal/shedadapters/... ./internal/lyxcwd/... ./cmd/lyx/...
```
