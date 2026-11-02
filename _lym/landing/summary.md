# Drift-Verify: operator-in-the-loop mode via verify_interactive

Drift-Verify judges drift alone and can accept a regression in a numerical code base, where the operator wants to decide how large a deviation is acceptable and in which quantities.
A new `verify_interactive` key in `drift.yaml` (default `false`) turns the last drift step into a session with the operator in its pane.
Drift-Baseline stays autonomous.

Closes #14

## What changes

- With `verify_interactive: true`, `VerifyTable` builds an interactive lead seat.
  Its prompt has the agent present the drift one test at a time: per quantity the baseline value, the current value and the deviation, then the baseline report's expectation and its own judgment.
  It discusses each test with the operator and records only the acceptances the operator decides with `lym drift accept`; tests the operator decides to fix go to the fix step.
- With the key `false`, the agent is autonomous as before.
  The verify stencil's mode and judging text are now the `{{.mode_rules}}` and `{{.judging_rules}}` markers, filled by Go from the mode; the autonomous text is unchanged.
- `verify_interactive` is in the `drift.yaml` template and the config spec, with no executable category.
- The prompt tells the agent to ask in plain text in the pane and never to call `AskUserQuestion`, whose modal dialog the run cannot drive.

## Design decisions

- **The operator's acceptance is its own kind.** `lym drift accept` in an operator session records `By: operator`, a new `AcceptedByOperator`.
  Under scope full it is `ruled` at once, because the operator is the ruling authority and the parent has nothing to rule on.
  The agent's autonomous acceptance stays `unruled`.
- **The operator session may re-accept a withdrawn test.**
  This is otherwise reserved to the parent at a decision halt.
- **The mode is recorded per spawn.**
  `ResetRuns` stores whether the session was spawned interactive in the runs record, and `VerbStatus.OperatorSession` reads it back through `OperatorSessionOpen`.
  It is taken from the table that actually spawns, so a resumed live agent keeps its spawned mode even if the config changed in between.
- The `PATTERN-parent-directive` pattern now names the Drift-Verify seat as a second interactive role next to the Discussion chair.
- The `lym drift accept` and `lym drift withdraw` help text describe the operator session.

## To check

- The test over the stencils that enforces the parent-directive pattern (`contracts/stencils/parentdirective_test.go`) still matches the pattern text for the second interactive role.
  Its comment was updated to name both roles, but the matching pattern itself should be read by hand against the new wording in `pattern/PATTERN-parent-directive.md`.
- `verbStatus` in `internal/driftcli/wiring.go` now reads the operator mark, and `internal/driftcli/wiring_test.go` pins it; check that the test covers an open operator session end to end, through `lym drift accept` in a worktree.
- Not run: a real interactive Drift-Verify session with an operator in the pane.
  The prompt text and the plain-text questioning were checked only through unit tests of the table and the verbs.
