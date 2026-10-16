# Fixer report — plan, round 1

## Changes

- **F1** (MEDIUM, scope), fixed as proposed in `_lyx/plan/03-open-permission-flag.md`.
  Premise confirmed: `TestOpenCmd_ModelAliasEngine` is the only test that runs `c.openCmd()`, and `internal/shuttlecli/open.go` turns a `c.runner.Open` error into `output.Err`; `Runner.Open` returns an `OpenLaunch` error unwrapped (`internal/shuttleengine/open.go`), so a scripted recorder error reaches the envelope with its text intact.
  Card 3's Test paragraph now gives `openRecorderEngine` an `err error` field that `OpenLaunch` returns beside the fixed launch after recording the spec.
  It adds the `--role explore --permission-mode auto` refusal row, whose `fake` recorder is scripted with an `invalid permission mode` error, and which asserts a not-ok envelope whose error holds that text, plus one recorded spec with mode `auto`.
  Every other row keeps asserting an ok envelope.
  The rewritten doc comment also says a refusal from `Open` reaches the error envelope.
  No target list changed.

## Disputed

None.

## Deferred

None.
