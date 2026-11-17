# Relevant files

- `internal/gatecli/cli.go` — the `lym gate test` command and its flags, where `--map` joins and the testmap command is registered
- `internal/gatecli/testmap.go` — the verb being removed: setup, slot, guard and per-test timeout and memory values
- `internal/gatecli/testmappackage.go` `mapPackage`, `runSideBySide`, `runTest`, `reduce`, `fillMapTests`, `writeMap` — the measure loop map mode keeps
- `internal/gatecli/testmapplan.go` — the incremental planning that goes
- `internal/gatecli/testmapaudit.go` — the audit round that goes
- `internal/gatecli/testmapwait.go` — the build-lock wait that goes
- `internal/gatecli/redundantmutants.go` `memoryCappedParallel`, `clampParallelToMemory` — the memory clamp of side-by-side runs
- `internal/gatecli/memoryguard.go` `guardOwnTests`, `startMemoryGuard` — the guard over lym's own tree
- `internal/gatecli/redundant.go` — the missing-map refusal whose way forward names the verb
- `internal/gatecli/doc.go` — the package doc naming the verb, its slots and its guard
- `internal/gateslot/doc.go` — the slot holder list naming the verb
- `internal/gateslot/template.yaml` — comments naming the verb
- `cmd/lym/daemonwakeups_test.go` — a table row naming the wait file
- `pattern/PATTERN-test-speed.md` — the redundancy-run paragraph and coverage-map section naming the verb
- `pattern/PATTERN-gate-slot.md` — the slot rules a map run follows
- `docs/research/speed/testmap-smart-build-2026-11-15/report.md` — cost measurements of builds and per-test processes
