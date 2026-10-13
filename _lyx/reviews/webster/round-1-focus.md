---
round: 1
focus:
  - "Cards 1-2, `internal/shuttleengine` wait/send/readiness and `claudeengine` turn progress: settle whether `TurnProgressUnknown` as the zero value keeps today's override for every engine or fake without `TurnProgressReader`, and whether the veto is lifted when a session start follows the unmatched turn start."
  - "Card 2, the transcript reader: settle whether the newest main-chain entry is chosen correctly with skipped, sidechain, null-`stop_reason`, thinking-only and no-trailing-newline lines, and whether the `testdata/turnprogress` rows each pin a distinct behavior."
  - "Card 3, fork authorization recorded at spawn: settle whether `RunState.ForkSubagents` is written once at spawn, read by the fork audit and by `Run.evaluateGate`, and whether an old run record without the field behaves as before."
  - "Cards 4-6, lens-review gate: settle whether `Plan-Write` is the only row whose plan gate carries the lens-review closure (via `planWriteEntry`, with no row-name switch in `resolveGateSpec`), `Final` keeps precedence, and `validate-plan --lens-review` matches the gate's check exactly, including flag refusals (`PATTERN-gate-self-check-parity`)."
  - "Card 7 and the per-card mechanical checks: settle whether the twill directive's step 8 names the `--lens-review` command for the Plan host only, and whether each card group's type-specific check ran against all its targets."
  - "Test economy across the diff: settle whether new tests in `gate_test.go`, `wait_test.go`, `run_test.go` and `twilldirective_test.go` duplicate existing coverage, and whether any added behavior has none."
---

Seed focus from the plan's card index and the diff stat; no prior round exists.
