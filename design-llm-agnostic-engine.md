# Make shuttle and lyx neutral to the LLM provider

`llm-agnostic-engine` · Note · enhancement, shuttle, config, stencil

## Idea (operator, 2026-10-13)

Everything about driving an LLM is meant to go through shuttle, so a new provider (Codex CLI first, `codex-engine`) should need only one more engine.
Rewrite shuttle and the other modules so that holds: lyx is neutral to the LLM, not tailored to Claude Code.

## Seen (feasibility study, 2026-10-13)

Report `.scratch/codex-feasibility.md` in the hub prime. Couplings outside the engine interface:

- The engine choice is hardwired at about 15 call sites (the `wiring.go` files of burlercli, webstercli and loomcli, `loomcli/looparm.go`, `orchcli/cli.go`, `battencli/wire.go`, `pausecli/deps.go`, `shuttlecli/cli.go`, `cliwire/sessionnamer.go`, `shuttleengine/rundir.go`).
  `modelspec` has an `Engine` field, but only `claude` is accepted and nothing reads the resolved engine.
- `run.json` records no engine, so readers over many runs assume one.
- `shuttleengine/hold.go`'s hold-notice text names `SendMessage`: a provider leak into the core.
- tokencount and the timing report read `~/.claude/projects` and carry Claude prices.
- Deploy syncs skills only into `~/.claude/plugins`.
- Stencils assume Claude Code features: Agent-tool forks, `SendMessage`/`ListAgents`, background Bash that wakes the session.
- Smaller: `CleanClaudeEnv` in reed, `claude_*` config keys, session naming, CLAUDE.md counted in Merriam's base.

## Scope to discuss

- An engine registry chosen per role through model specs, the engine recorded in `run.json`, multi-run readers that pick the reader per run.
- Provider-neutral wording in core notices; provider-specific text only in the engine.
- Usage and transcript readers behind the engine, so tokencount and timing work for any engine.
- Which stencil features get a neutral form (a parent directive that does not rely on `SendMessage`) and which stay Claude-only, refused for other engines.
