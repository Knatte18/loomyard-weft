# PATTERN: tests never fake another module with made-up data

`no-fake-other-modules` · Note · enhancement, pattern, testkit

## Operator decision (2026-11-26)

Tests never use fabricated stand-in data that simulates another real module: a hand-written fake of another module's state, records, config or output.
Such a fake goes stale the moment the real module changes, and the test keeps passing against a module that no longer exists.
A test that crosses into another module drives the real module, or the test belongs in that module.
A module's own state built by hand for its own rules is not a stand-in.

## What to do

- Write it as a PATTERN under `pattern/`.
- It conflicts with what stands now; settle each:
  - `PATTERN-test-speed.md` says a behaviour testable with a fake or an in-memory fixture is tested at tier0 that way.
  - `PATTERN-test-isolation.md` describes shared fake kits, such as `shedfake`, that stand in for other modules.
- List the existing fakes of other modules across the repository and say for each whether it goes, becomes the real module, or is a module's own state.
  The sweep itself may be follow-up tasks.
