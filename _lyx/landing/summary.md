# Research: audit Claude-specific stencil mechanics for a second engine

Adds `docs/research/stencil-engine-audit-2026-10-24/report.md`, a read-only audit of every stencil, spec and prompt-inlining Go site that depends on a Claude Code mechanic, sized for the `codex-engine` note.
Nothing outside the new folder changes.

The report groups the sites by mechanic into thirteen proposed include blocks, following the operator's decision that each engine-specific mechanic becomes one named block an engine supplies.
Each block gives a proposed name, the Claude text it would hold, the sites (file and line) and what Codex needs there, citing only `docs/research/codex-spike-2026-10-24/report.md`.
A mechanic the spike did not exercise is marked untested, and one it found no counterpart for is marked as having none.

Findings a reviewer will want first:

- The fork spawn (blocks `spawn-fork-fan`, `spawn-batch-fork`, `spawn-oneshot-fork`, `fork-inheritance`, `no-nested-spawn`) has a Codex counterpart in `spawn_agent` with `fork_turns` set to `all`; only the enforcement of the nesting ban is untested.
- The wait on a background completion (`await-completion`) has only a partial counterpart, and the webster Master, the shed driver and the orch role all rest on it.
- The agent-to-agent message (`message-agent`) has no counterpart in the spike.
- Context cycling commands (`session-lifecycle`) are untested.
- The include mechanism (`stencil.FillWith`) already exists; what is missing is an engine-keyed choice of include set at render time.

The size estimate ends the report: 13 blocks, 127 stencil sites in 45 of 83 stencils, 8 spec sites in 6 of 11 specs, and 35 Go sites.

## Design decisions

- Sites are grouped by mechanic rather than by file, so the table shows how many stencils a single block would touch.
- The word "fork" used only as a lyx role name is not listed as a site, since it holds while a fork exists on the engine.
- Counts are row counts: a site that depends on two mechanics appears under each.
- No test was run: the change is a markdown file under `docs/research/` that no build or test reads, and it carries no inline links.
