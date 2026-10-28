```yaml
lens: code-affected-set
outcome: reported
```

The lens asked for a list per change, the set found, covered or not. I found no defects, so there are no findings.

- **`Deps.TaskLimit` field.**
  - Set found: `Deps` struct literals in `beamcli/dobbydeps.go:93` (filled with `proc.ReadTaskLimit`), `dobbyengine/passes_test.go:144` (fake seam filled), `loop_test.go:125` and `tidy_test.go:66`.
  - Covered: the last two leave the seam nil, which the nil-seam path accepts. `dobbytrace_integration_test.go` and `hubgeom/dobbygeom.go:88` build `Deps{}` or `StartDeps`, not this struct.
- **beam.yaml key `task_headroom`.**
  - Set found: `Config` field, `ConfigSpec` key map, the non-negative key loop in `LoadConfig`, `template.yaml`, and `config_test.go` (default 500 and negative refused).
  - Covered: `doc.go` mentions no list of all keys, and no other doc or fixture names `board_tidy_every` beyond these, so no other key list needs the new one.
- **Start-step behaviour (hold, log lines, `taskLimitHeld` episode state).**
  - Set found: `starts.go` and `loop.go`, no other writer or reader of `taskLimitHeld`, no other file asserting "starts resume".
  - Covered: `doc.go` states the hold, and its view sentence ("starts on the dobby's next tick") is bounded. `starts_test.go` asserts the lines.
- **`proc.ReadTaskLimit` and `TaskLimit`.**
  - Set found: `proc/doc.go` names the reader, and `beamcli` is the only production caller.
  - Covered: `loop.go` already imported `proc`, and `starts.go` adds the same import. No new import edge for dobbyengine, so `PATTERN-cwd-resolution`'s seam rule holds.
- **PATTERN.md and commit message.**
  - PATTERN: the diff adds no new cross-cutting invariant. `PATTERN-test-isolation` and `PATTERN-tolerant-records` hold (temp-dir fixture, no state-record change).
  - Commit message: one commit, no `Pattern-Departure:` trailer, and none is needed.
  - Declared PATTERN changes: the directive declares none.
  - Stencils: none changed.

Tool calls (2):
1. Grep for `board_tidy_timeout_min` and `Deps{` construction sites, and the `proc` imports in `loop.go` and `dobbydeps.go`. I needed the full set of key-list and `Deps` sites, which the manifest lacked.
2. Grep for other `board_tidy_every` mentions and for `taskLimitHeld` and "starts resume" outside `starts.go`. I needed to know whether another doc or test asserts them.
