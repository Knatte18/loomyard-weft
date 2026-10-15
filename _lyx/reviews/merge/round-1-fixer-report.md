# Fixer report — merge, round 1

## Changes

- F1 (LOW, fixed with own change): `internal/shuttleengine/enginedispatch_test.go` gains the subtest `Send_and_Interrupt_on_a_live_strand_with_no_record_consult_the_default` in `TestRunner_EngineSetDispatch`.
  It builds the dispatch rig with no run record and a live strand in reed, calls `Runner.Send` and `Runner.Interrupt`, and asserts through `requireConsulted` that engine a (the default) was consulted and engine b recorded nothing.
  Differences from the proposed fix:
  - The review proposed rows in the subtest `a_record_with_no_engine_key_is_driven_through_the_default`, which has a run record via `withStrand`; a recordless case needs its own subtest.
  - `SessionIdle` is not added: the existing subtest `SessionIdle_on_a_live_strand_with_no_record_consults_the_default` already pins it.
  - The test pins only the consulted engine, not the call's error: the fake pane never echoes the sent text, so `Send` returns a not-landed error, as in the existing dispatch table where `Runner.Send` has no outcome assertion.

`lyx gate test ./internal/shuttleengine` is green.

- Gate finding (PATTERN.md headroom, 47 bytes over): shortened the `PATTERN-creel-store` line in `PATTERN.md` to 15564 bytes, the headroom line.
  The full wording stays in `pattern/PATTERN-creel-store.md`, which is unchanged.
  `lyx gate test ./internal/pattern` is green.

## Disputed

None.

## Deferred

None.
