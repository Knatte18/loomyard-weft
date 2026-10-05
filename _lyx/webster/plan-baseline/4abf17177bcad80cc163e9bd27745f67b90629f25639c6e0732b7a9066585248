---
format: 5
approved: true
language: go
---

# Plan: test-suite-measure

Measure the Go test suite per test, speed up the integration tier through test seams and parallelism, stop test runs leaking tmux socket files, split the tag tiers into `integration`, `tmux` and `llm` so no tier below `llm` spawns an LLM, and consolidate PATTERN's Testing section into five entries.
The plan measures the before-state and the after-speed state on the same pre-retag test set (D4), then retags, fixes the socket leak, and ends with the per-test redundancy tool and its committed report, which `test-suite-prune-core` and `test-suite-prune-rest` consume.
No test is deleted.

## Card Index

1 — testtiming-package-stats — testtiming reports per-package test count and serial time, and takes `-tags`
2 — timing-before-state — record the dated before-state with its slowest-tests table
3 — watchdog-timing-seam — inject the watchdog daemon's timing per command instance
4 — reap-timeout-seam — inject the reap timeout into the watchdog's session reap
5 — verify-wait-seams — shorten verifyrun's wait delay and verifytree's sleeps in tests
6 — fabricengine-parallel — run fabricengine's serial integration tests in parallel
7 — vocab-prefilter — prefilter the bare weft/warp scan before its regexes
8 — gitoracle-hermetic — run gitoracle's tests under the hermetic git environment
9 — timing-after-speed — record the after-speed timing on the pre-retag test set
10 — pattern-testing-consolidation — merge the Testing entries into five and rename every retired ID
11 — llmkit — add the `llmkit` kit and its bounded `os/exec` exemption
12 — guards-know-tier-tags — teach the tier guards the `tmux` and `llm` tags
13 — retag-smoke — retire `smoke`: its files move to `tmux` or `llm`, onto `llmkit`
14 — retag-integration-tmux — move tmux-starting `integration` files to `tmux`
15 — llm-spawn-guard — guard LLM spawns into the `llm` tier
16 — tier-docs-and-stencils — document the four tiers in docs, spec and stencils
17 — tmuxkit-socket-cleanup — tmuxkit removes the socket files of the servers it kills
18 — test-socket-keys — move ad hoc tmux keys in tests onto tmuxkit
19 — reed-socket-removal — reed removes the socket file of a server it tore down
20 — testtiming-redundancy — testtiming's per-test coverage redundancy mode
21 — redundancy-report-and-post-retag — commit the redundancy report and the post-retag timing

## Shared Decisions

- **Measurement order (D4).** Cards 2 and 9 measure on the same pre-retag test set, before and after the speed fixes (cards 3–8); no card before 9 adds, removes or retags a tagged test file.
  The untagged files cards 1 and 8 create are in both measured runs.
  Card 21 records the post-retag numbers separately.
- **Seams are per instance (D5).** A shortened wait is a struct field or a parameter, never a package-level `var`, so tests keep `t.Parallel`; production always passes today's value.
- **PATTERN lands with its code.** Card 10 merges the Testing entries on today's rules; each later rule (the `llmkit` exemption, the tier table, the LLM guard, the socket rule) lands in `PATTERN.md` and its background in the same card as the code that enforces it.
- **Shared helpers across tiers.** A test helper file used by files of two tiers and spawning something carries the disjunction of their tags (`//go:build tmux || llm`), recorded after Discussion; a helper that spawns nothing goes untagged.
- **File names stay.** Retagged `smoke_*_test.go` files keep their names; `internal/shuttlecli/modelpin_test.go` and the crucible docs find them by that prefix.
- **Tier detection.** A file belongs to `tmux` when its tests start a tmux server: the plan detects them by running the integration tier with `tmux` absent from `PATH`; a test that only probes `tmux has-session` stays `integration`.
- **Gate selection is out of scope.** `cardGateCommand` is unchanged, so after cards 13–14 the card gates run no `tmux` test; this plan's verify runs `-tags tmux` (D6), and `llm` files are compiled only.

## verify:

```
go build ./...
go vet -tags integration,tmux,llm ./...
go vet -tags integration ./...
go vet -tags tmux ./...
go vet -tags llm ./...
go test ./...
go test -tags integration ./...
go test -tags tmux ./...
```
