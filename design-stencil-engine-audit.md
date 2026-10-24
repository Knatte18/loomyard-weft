# Audit the Claude-specific mechanics in stencils for a second engine

`stencil-engine-audit` · Task · audit, stencil, codex · done

## Seen (2026-10-24)

Most stencil text says what to review or write and is engine-neutral; the Claude-specific parts are mechanics: how a fork is spawned (the Agent tool with `subagent_type: "fork"`), `SendMessage`, skill loading, Claude tool names.
Stencils render through Go `text/template` (`internal/stencil/stencil.go`), which supports `{{if}}` and named includes through `{{template "name"}}` (`FillWith`).

## Idea

Audit every stencil and spec for Claude-specific mechanics and list each site with what it needs on Codex, to size the rewrite.
Decision (operator, 2026-10-24): each engine-specific mechanic becomes one named include block (such as `spawn-fork`) that each engine supplies, so a stencil keeps one text instead of `{{if}}` branches at every site.

## Task (operator, 2026-10-24)

A report only, nothing built: `docs/research/stencil-engine-audit-2026-10-24/report.md`.
- Read every file under `contracts/stencils/` and `contracts/specs/`, and the Go code that renders or inlines prompt text Claude-specific (such as shuttle's prompt assembly and `internal/claudeengine`), read-only.
- List each Claude-specific site: file and line, the mechanic (fork spawn, `SendMessage`, skill loading, a Claude tool name, background-task wake-up, Claude settings), and what Codex needs there, from `docs/research/codex-spike-2026-10-24/report.md`; mark a site with no Codex counterpart.
- Group the sites by mechanic into the named blocks the decision above calls for, with a proposed block name and the Claude text each would hold.
- End with a size estimate: blocks, sites, and stencils touched.
- The run changes nothing outside the new folder.
