# Fixer-report, round 1

## Changes

- F1 (fixed as proposed): `internal/shuttleengine/session_test.go` gains a `seeded` field in the `SessionIdle` table, whose length sets the run's `PromptOffset`, and three rows that drive `Runner.SessionIdle` through `sessionReadiness`:
  - a non-zero offset with a session start, turn start and turn end before it reads the load turn's end;
  - a non-zero offset with an unmatched turn start before it stays held with "a turn is running";
  - a zero offset with an empty events file reads unknown and takes the pane fallback.
- Commit c448dbf2: `shuttleengine: fix F1 — pin the readiness fallback through SessionIdle`.
- Self-check finding (fix-consistency, LOW): the test function's doc comment now names the prompt-offset rows.

The gate `lym gate test ./internal/shuttleengine` passed before the self-check: tier0, 1280 passed, 0 failed.

## Gate repair

`TestGofmt_ModuleFilesAreFormatted` failed because the F1 edit dropped the space after a row's `name` string (`one",probes:`).
`lym loom fmt` restored it, in a third commit.

## Self-check

- fix-consistency, LOW: the doc comment of `TestRunner_SessionIdle_ReturnsScriptedClassification` omitted the prompt-offset rows. Fixed as proposed.
- fix-code-correctness: no findings.
- fix-proposal: no findings.
- fix-affected-set: no findings.

## Disputed

None.

## Deferred

None.
