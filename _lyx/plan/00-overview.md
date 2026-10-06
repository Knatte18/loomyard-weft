---
format: 5
approved: true
language: go
---

# Plan: load every skill in one turn

A spawned role, and the orch after each cycle, loads its whole skill list in one model turn: one plain typed message asks the model to load each skill through the Skill tool, and the turn end is checked against the transcript.
Unknown skills are skipped at once with a Warn, missing ones are retried once, an unreadable transcript confirms the load unverified, and a load turn that never ends is skipped at the load timeout, so an unknown skill never stalls a launch or a reload (issue #394).
Card 1 adds the Claude message and transcript check, card 2 moves shuttle's launch onto them, card 3 moves the orch reload, and card 4 removes the per-skill load and the pane unknown-skill classifier.

## Card Index

1 — skill-load-transcript — add SkillLoadReport and Claude's one-message load and transcript classification
2 — launch-skills-one-turn — shuttle's launch loads the list in one turn with one retry; add the per-list Runner primitives
3 — orch-reload-one-turn — the orch reload becomes skills step, optional retry step, pointer step
4 — drop-per-skill-load — remove the `/<skill>` load and the pane unknown-skill classifier

## Shared Decisions

### Decision: provider-neutral-seam

- **Decision:** `shuttleengine` and `orchengine` pass only skill names and a `SkillLoadReport`; the message text, the Skill-tool name and every transcript shape live in `claudeengine/skillload.go`.
- **Rationale:** `PATTERN-shuttle-provider-seam`; the message is engine choreography like `/compact`, not a stencil.
- **Applies to:** all cards

### Decision: warn-vocabulary

- **Decision:** a skipped skill is one `logger.Warn` per skill with keys `skill` and `cause`, the cause one of `unknown`, `not loaded` or `timeout`; an unverified turn is one `skill load unverified` Warn listing its skills.
  Shuttle logs under `shuttle:` and orch under `orch:`, as today.
- **Rationale:** one vocabulary across launch and reload, through `internal/logger` (`PATTERN-spawn-observability`).
- **Applies to:** cards 2 and 3

## verify:

go build ./...
go test ./internal/shuttleengine/... ./internal/testkit/shuttlefake/... ./internal/orchengine/... ./internal/orchcli/... ./internal/loomcli/...
go test -tags integration ./internal/shuttleengine/... ./internal/testkit/shuttlefake/... ./internal/orchengine/... ./internal/orchcli/... ./internal/loomcli/...
go test -tags tmux ./internal/shuttleengine/... ./internal/testkit/shuttlefake/... ./internal/orchengine/... ./internal/orchcli/... ./internal/loomcli/...
go vet -tags llm ./internal/shuttleengine/... ./internal/testkit/shuttlefake/... ./internal/orchengine/... ./internal/orchcli/... ./internal/loomcli/...
