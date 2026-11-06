# Fixer-report — code round 1

## Changes

Each finding is fixed as proposed unless marked; each fix is its own commit.

- **F1** (`6bd4a7a2`): `internal/loomcli/arm.go` declares `driftBaselineEngine = "DriftBaseline"` beside `darnWriteEngine`, and `wiring.go` passes it to `loomrecipe.HasEngine`.
- **F2** and **F3** (`0fa7256f`): `internal/driftengine/diffmode.go`.
  - `skipFileArgument` takes a quote around the assignment only as a matched pair, through three alternatives (double-quoted, single-quoted, unquoted), so the quote closing an enclosing `sh -c '...'` string is never taken and such a command stays whole.
  - `argumentRemainder` replaces the `$1` replacement: an argument whose leading whitespace holds a newline keeps that line break and drops the whitespace after it, so a `\`-continued command keeps its continuations.
  - `TestMigrateConfig_SkipFileArgument` builds its document with `yaml.Marshal`, so a command may hold quotes and newlines, and gains three rows: the `\`-continued command, and an enclosing single-quoted and double-quoted shell string; a row whose want equals its command expects no line.
- **F4** (`ae2a6fff`): `verify_test.go` gains a `checkoutsErr` fixture field and a row asserting that a failing snapshot on a fresh judge session leaves the `judge_session` mark closed and spawns no judge.
- **F5** (`d915db64`): `internal/configreg/runsettings_test.go` gains a refusal row each for `volatile_files` and `volatile_keys` naming the hub's drift.yaml (the guard-key rows live there, not in `driftengine/config_test.go` as the review's location key said).
- **F6** (`346b4959`): the `ARecordWithNoDiffRecordsTheConfigs` row of `expect_test.go` sets `Volatile` in the config and asserts the recorded scope holds it.
- Self-check follow-up (uncommitted at the time of writing, committed after): the `MigrateConfig` doc comment and `internal/driftengine/doc.go` now say that an argument inside a quoted shell string stays for the operator.

## Self-check

- fix-code-correctness: no findings.
- fix-consistency, LOW: `MigrateConfig` doc comment and `doc.go` say "each whole argument" without the quoted-shell-string exception; fixed as proposed.
- fix-proposal: no findings.
- fix-affected-set, LOW: bare `"PlanWrite"` (`decision.go:1036`) and `"Implementer"` (`wiring.go:661`) engine names remain in `loomcli`; not a finding, since the review never asked for them and the fix did not make them worse (rubric: a further improvement the fix did not worsen), so left as they were.

All four forks returned a report.

## Disputed

None.

## Deferred

None.

Gate: `lym gate test ./internal/driftengine ./internal/configreg ./internal/loomcli` passed before the doc edits (0 failed).
