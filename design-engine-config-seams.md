# Engine-owned config, one neutral guardrail table, provider-blind reed

`engine-config-seams` · Task · enhancement, shuttle, claude, config, reed · running · Publish

## Goal

Second task of the Codex-readiness series (see `engine-registry` for the operator decision): engine settings owned by each engine, one neutral guardrail table, and reed blind to the provider.

## Seen (2026-10-15)

- `shuttleengine/config.go` and its `template.yaml` hold Claude keys: `claude`, `claude_deny_agent_tool`, `claude_deny_ask_user_question`, `claude_deny_python`, `claude_prompt_cache_ttl(_roles)`.
- The guardrail table in `claudeengine/settings.go` mixes neutral shell policy (python, raw go and gate-slot, `sed -i`, computed `rm`, sleep) with Claude tool denies (Agent, AskUserQuestion, webster-in-fork); `cmd/lyx/norecoverydeny_test.go` keeps a hand list mirroring it.
- `reedengine/env.go` `CleanClaudeEnv` is used at reed's launch path.
- reed's `SessionNamer` seam is filled once for every strand (`reedcli/watchdog.go`), with no per-strand provider.

## Scope

- A `claude.yaml` module owned and loaded by `claudeengine`, registered in `configreg` and layered (default, worktree, local): binary path, prompt-cache TTL and per-role map, the Agent and AskUserQuestion denies.
- The neutral switch `deny_python` stays in `shuttle.yaml`; migration moves the old keys out of `shuttle.yaml`.
- One neutral shell-guard table (grep pattern, steer, notice, switch) in a leaf package; `claudeengine` renders it as PreToolUse hooks, and the no-denied-recovery test reads the table instead of its hand list.
- Engines declare the environment keys to scrub, replacing `CleanClaudeEnv`.
- reed strands carry an opaque provider tag set by shuttle; the watchdog takes a `SessionNamer` per provider, and an engine with none gets no session-name repair.

## Out

Codex hook rendering and `codex.yaml` (note `codex-engine`).

## Design points: the planning agent's recommendations, not yet decided by the operator

- A module file per engine (`claude.yaml`, later `codex.yaml`) rather than an `engines:` map in `shuttle.yaml`, matching the rule that settings live in the owning module's config with layered overrides.
- Guardrails as a neutral grep table each engine wraps in its own hooks, not a `lyx` hook verb (which would start a `lyx` process on every Bash call; see note `spawn-audit-log`).
- Per-engine overrides of timing keys such as `submit_settle_ms` wait for the Codex tuning.

Note `spawn-audit-log` (reed launch path) should land after this task if promoted.
