---
format: 5
approved: false
language: go
---

# Plan: engine-registry

Shuttle picks each run's engine by name from a registry and records it in the run record, so a later engine plugs in at `internal/shuttleengines` and one `modelspec` name with no change at any shuttle caller.
The `Engine` interface shrinks to what every engine must do: fork audit becomes an optional capability, `ModelSwitchSequence` is deleted, and requirement support, resume-context rendering and instruction files join as capabilities.
The cards build the set and its config key first, then the registry, then move the readers, the runner constructors and the per-run operations onto it, before the requirement checks, the capability moves and the seam scan; each card carries the docs its change makes wrong.

## Card Index

1 — engine-set — add EngineSet with its lookup and the shuttlefake set helper
2 — engine-config-key — add shuttle.yaml's engine: key, refusing an empty value
3 — engine-capabilities — declare Requirement and the optional capabilities, implemented by Claude
4 — model-choice — add ModelChoice and export modelspec's engine names
5 — shuttleengines-registry — add the registry package and move SessionNamer into it
6 — cross-run-readers — dispatch the three readers per run record through the set
7 — runner-engine-set — make the Runner constructors take the set at every call site
8 — per-run-engine — resolve Start's and Open's engine by spec and every other operation's by record
9 — guid-engine-reads — give InterruptSettle and the guid-less session reads a guid, through orch and tender
10 — spec-requires — check Spec.Requires at Start and attach
11 — fork-audit-capability — move fork audit off Engine and assert it in Webster's wiring
12 — orch-resume-context — render the orch resume context through the engine capability
13 — model-switch-delete — delete ModelSwitchSequence everywhere
14 — provider-seam-scan — amend PATTERN-shuttle-provider-seam and widen its scan

## verify:

go build ./...
go vet -tags integration ./...
go vet -tags tmux ./...
go vet -tags llm ./...
go test ./...
go test -tags integration ./...
