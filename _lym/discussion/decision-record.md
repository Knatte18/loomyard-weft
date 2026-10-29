# Decision record — verifygate-misplaced-guard-test

## Goal

Cover the verify gate's guard scan over a narrowed diff again, now that the gate key's computed scope reads its diff base from the green record.

## Scope

- Add back the subtest `fails with the file and line of a misplaced marker` to `TestVerifyGate_RoundScenario` in `internal/loomshed/verifygate_integration_test.go`, with no skip.
- Test-only: no production code changes.

## Decisions

### D1 — Restore the subtest at its old place in the round scenario

The subtest commits `a/a_test.go` with `//lym:guard` above `var misplaced = 1` on line 5, then runs the round form with no merge base reader and the darn form with the first recorded pass as merge base and a not-scoped tree scope.
Both forms must fail with findings naming `a/a_test.go:5` and the way to place the marker.
It sits after the darn-form read-error table and before `fails on a wrapped new comment before any test runs`, whose first commit rewrites `a/a_test.go` to a clean file, so later steps see no misplaced marker.

Rationale: the told command is the `code-burler-fix` key's call, a computed scope whose base is the newest whole-green ancestor in the green record, so the guard scan now runs over the narrowed diff and its refusal reaches the findings through the log tail.

### D2 — No doc change

`internal/loomshed/verifygate.go` already documents that a misplaced `//lym:guard` marker reaches the findings with its file and line, and `internal/gatecli/doc.go` already documents the base read from the green record.

Rationale: the change adds coverage only; no observable behavior or invariant changes.

## Constraints

- `PATTERN-test-speed`: the subtest keeps the file's `tier1` tag and skips nothing.
- `PATTERN-test-isolation`: the fixture stays the scenario's hermetic scratch repo.

## Auto-mode assumptions

- The guard refusal's text "directly above" is a stable part of the way forward that `internal/gatecli/cli.go` prints.

## Open risks

- Should the computed scope fall back to whole for the step, the guard scan still runs over the module, so the subtest still holds; a scope that skipped the scan entirely would fail it, which is the regression it guards.

## Acceptance criteria

- D1: the round form fails naming `a/a_test.go:5` and "directly above".
- D1: the darn form told a not-scoped answer fails naming `a/a_test.go:5` and "directly above".
- D1: the later steps of `TestVerifyGate_RoundScenario` still pass, and `lym gate test` is green over the change.
- D2: no doc file changes.
