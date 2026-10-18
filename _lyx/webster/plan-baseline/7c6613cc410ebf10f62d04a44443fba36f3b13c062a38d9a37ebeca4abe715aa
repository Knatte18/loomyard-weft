---
format: 5
approved: true
language: go
---

# Plan: orch-rules-files

The orch's hub-specific rules leave the code repository.
The orch role file inlines two rules files in place of `ORCHESTRATOR.md`, hub-wide first: the hub's rules for every orch on Bolt at `<BoardDir>/_lyx/orch/rules.md`, and the orch thread's own rules in its worktree's weft at `<anchor>/_lyx/orch/rules.md`.
A new verb, `lyx orch rules --from <file>`, writes, commits and pushes the hub-wide file on Bolt; `ORCHESTRATOR.md` is deleted with its mentions, and a board note carries the operator's step to move today's rules after the deploy.

## Card Index

1 — rule-files-render — told hub-wide, own and prime-root paths on `orchengine.Paths`, and the role file's two rules sections
2 — orch-rules-verb — `lyx orch rules --from <file>` writes, commits and pushes the hub-wide rules file on Bolt
3 — orchestrator-md-removal — delete `ORCHESTRATOR.md` and reword `CLAUDE.md` and the role stencil's placement sentences
4 — orch-rules-move-note — file the `orch-rules-move` board note holding D4's operator step and the rules verbatim

## Shared Decisions

- `lyx orch rules` keeps the `orch` group's `PersistentPreRunE` as it is, engine config loads included, rather than a `rules` branch that skips them: every orch verb then shares one refusal order, with the prime-only refusal first, and a hub whose orch config fails to load is refused by the config loader's own way forward before any Bolt write.
- The rules file's relative path, `_lyx/orch/rules.md`, is joined once in `rulesRel` (card 1); `orchPaths` roots it at the board dir and at the anchor, and the verb commits it as the one Bolt-relative path.
- The verb's Bolt calls go through a `rulesBolt` seam whose fake drives every failure branch in a unit test; one `hubforge` hub test pins the real commit and push.

## verify:

go build ./cmd/lyx
lyx gate test ./internal/orchengine ./internal/orchcli ./contracts/stencils
lyx gate test --tags integration ./internal/orchengine ./internal/orchcli ./contracts/stencils
