# Audit the Claude-specific mechanics in stencils for a second engine

`stencil-engine-audit` · Note · audit, stencil, codex

## Seen (2026-10-24)

Most stencil text says what to review or write and is engine-neutral; the Claude-specific parts are mechanics: how a fork is spawned (the Agent tool with `subagent_type: "fork"`), `SendMessage`, skill loading, Claude tool names.
Stencils render through Go `text/template` (`internal/stencil/stencil.go`), which supports `{{if}}` and named includes through `{{template "name"}}` (`FillWith`).

## Idea

Audit every stencil and spec for Claude-specific mechanics and list each site with what it needs on Codex, to size the rewrite.
Operator view, not a decision: the engine-specific parts become conditional on the engine.
Orch recommendation, not a decision: each mechanic becomes one named include block (such as `spawn-fork`) that each engine supplies, so a stencil keeps one text instead of `{{if}}` branches at every site.
