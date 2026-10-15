---
format: 5
approved: false
language: go
---

# Plan: engine-owned config, one neutral guardrail table, provider-blind reed

The second task of the Codex-readiness series.
`claudeengine` gains its own layered `claude.yaml`, registered in `configreg` through `internal/shuttleengines` and loaded lazily where the engine reads it; `shuttle.yaml` loses every Claude key, keeps the python switch as `deny_python`, and declares the moved keys retired, while a declared key move carries the old values into `claude.yaml` at every reconcile layer and in run settings.
The shell guards move into the new leaf `internal/shellguard`, which `claudeengine` renders as its PreToolUse hooks and the no-denied-recovery scan reads.
reed stops carrying Claude knowledge: it scrubs the environment keys the engines declare, and its session-name repair picks a namer per strand by an opaque provider tag shuttle sets.
Claude stays the only registered engine, and a golden test pins every Claude settings document and notice byte for byte before the refactor starts.

## Card Index

1 — settings-golden — pin today's settings document and deny notice for every deny-input combination
2 — shell-guard-table — create `internal/shellguard` and render its rows in `claudeengine`
3 — denied-forms-from-table — the no-denied-recovery scan reads the table's forms
4 — retired-config-keys — `configengine.Spec` retired keys: admitted locally by category, dropped by the loader
5 — claude-config-module — declare `claude.yaml` in `claudeengine` and register it through `shuttleengines`
6 — engine-loads-claude-config — the engine loads `claude.yaml` from told `Dirs`; `shuttleengines.New` takes them
7 — shuttle-drops-claude-keys — `shuttle.yaml` loses the Claude keys, retires them and renames `deny_python`
8 — config-key-move — the declared `shuttle` to `claude` key move in every reconcile and in run settings
9 — engine-scrub-keys — engines declare the environment keys reed's server spawn scrubs
10 — strand-provider-tag — strands carry an opaque provider tag; the watchdog picks a namer per tag
11 — config-package-docs — `configreg` and `configsync` package docs

## Shared Decisions

- `claudeengine.New()` keeps its signature and zero `Dirs` for reading uses; only `NewFromConfig` engines, built by `shuttleengines` with told `Dirs`, run `Prepare`, `OpenLaunch` and `PromptCacheLifetime`.
- `internal/shuttleengines` is the only route from a concrete engine to `configreg` (config modules), reed (scrub keys) and the watchdog (session namers).

## verify:

```
go build ./...
go vet -tags integration ./...
go vet -tags tmux ./...
go vet -tags llm ./...
go test ./...
go test -tags integration ./...
```
