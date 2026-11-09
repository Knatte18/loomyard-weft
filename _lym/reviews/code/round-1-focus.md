---
round: 1
focus:
  - "Scope: `internal/orchcli/{cli.go,stop.go,doc.go,beamdown_test.go}` is outside the record's three packages; settle whether the Goal's 'ended on stop through runproc.EndRole' licenses it, or whether the stop belongs in dobbyengine/beamcli."
  - "Trigger: `internal/dobbyengine/testmap.go` and `tidy.go` must count landings as the board tidy does and fire only after every `test_map_every` landings; settle that 0 never refreshes and that no prime-HEAD record or exit-record trigger remains."
  - "Single-flight: `internal/beamcli/dobbydeps.go` must start no refresh while a live `runproc.RoleTestMap` record exists under `runproc.HubOwner`, including after a dobby restart; settle how liveness is read and what a stale record does."
  - "Stop: settle that the child and the processes below it end through `runproc.EndRole`, on the prime dobby's stop only, and that a failure there is surfaced and not dropped."
  - "Config: `config.go`, `template.yaml` and `config_test.go` must add `test_map_every` and `test_map_batch` with defaults, load through `configengine`, change no existing key's meaning, and add no `test_map` on/off key; settle the batch default and the `--max` argument passed."
  - "Prime-only gating: settle that the step runs only on the dobby whose worktree carries the prime parent branch, and that the spawn uses the binary `selfexec.LymBinary` names."
  - "Docs: `docs/overview.md` and the package `doc.go` files must match the shipped behaviour; check no mention of the superseded D6 trigger (prime-HEAD record, `test_map` key) survives."
  - "Test economy: `testmap_test.go`, `dobby_test.go` and `beamdown_test.go` for duplicated coverage and for uncovered behaviour (restart with live record, cadence of 0, stop ending the role); no test may launch a test binary or `go test`."
  - "Summary and commits: check claims in `_lym/landing/summary.md` against the diff, and the `Pattern-Departure:` trailers, if any, across the three commits."
---

Seeded from the decision record's Goal, which supersedes D6's trigger, and from the diff stat of the review range.
