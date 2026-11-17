# Warn on resume when a run's stencils changed since it started

`stencil-pin-per-run` · Task · feature, stencil, shed · running · Drift-Verify

## Seen (orch, 2026-11-17)

Runs last hours to days and stencils change often, so a resumed run's later producers may get prompts that differ from those it started with,
and nothing says so.
Source: the survey F17 (`orchestrator-survey-ideas`), from Conductor's `bundle build` and Bernstein, which refuses to resume after its spec digest changes.

## Idea

- A run records the digest of each stencil it may render when it starts, in weft.
- Resume compares them with the installed stencils and warns the orch of each that changed, naming `lym stencil diff`.
