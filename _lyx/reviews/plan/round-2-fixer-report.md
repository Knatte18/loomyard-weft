# Plan-Review round 2 — fixer report

Every finding in `round-2-review.md` is fixed.
`lyx loom validate-plan` passes over the edited plan (`ok: true`, informational `create-new-unit` findings only).
The overview's `approved:` key is untouched.

## Changes

- **F1 (MEDIUM), float values pass card 2's unmarshal.**
  Card 2's `Load` now decodes each key into a `yaml.Node` and rejects any value that is not a scalar tagged `!!int`, naming the key, before its zero/negative check.
  The card says why: yaml.v3 truncates a `!!float` into an `int` field without error.
  The test list now names a float (`1.5` and `14.0`) and a string case for each key.
- **F2 (LOW), card 5's fixtures.**
  Card 5 states that every `logger.yaml` fixture carries both keys: the count-2 and count-0 integration cases set `trace_retention_days: 14`, and the valid `exitsweep_test.go` case carries both keys at non-default values.
  An invalid fixture is defined once as a two-key file with `trace_retention_count: 0`.
  The count-0 integration case also asserts that its Warn line names `trace_retention_count`.
- **F3 (LOW), card 2's rejection tests.**
  Each rejection fixture is now a valid two-key file with exactly the key under test changed, and each case asserts that the error names that key.
  The valid-file case carries both keys at non-default values.

## Deferred

None.
