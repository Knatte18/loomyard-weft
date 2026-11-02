# Drift-Verify with the operator in the loop

In a numerical code base, deciding whether a drift is acceptable is the operator's call.
An autonomous Drift-Verify agent that accepts drift can approve a regression without anyone seeing it.
This change adds a hub-wide `drift.yaml` key, `verify_interactive`, which works like loom's `discussion_interactive`.
When the key is set, the Drift-Verify agent runs as an interactive session with the operator in the task's worktree pane.
It presents the drift per test and per quantity, together with the baseline agent's expectation, discusses each test with the operator, and records each acceptance or fix the operator decides before the run goes on.
The default is `false`, which keeps the autonomous behavior.
Drift-Baseline stays autonomous in both modes.

## What changes

- **Config** (`internal/driftengine/config.go`, `template.yaml`): adds `verify_interactive`, a plain boolean that is open to the local layer like the other non-executable drift keys.
- **Seat table** (`internal/driftengine/seats.go`): `VerifyTable` sets `Table.Interactive` from the key.
  It also fills two new stencil markers, `mode_rules` and `judging_rules`, with either the autonomous text or the operator text.
  The operator text has the agent show, for each drifted quantity, the baseline value, the current value and the size of the deviation, then the baseline report's expectation and its own judgment.
  It waits for the operator's decision on each test, accepts a test only on that decision with the operator's reason, fixes what the operator decides to fix, and asks in plain text in the pane, never through `AskUserQuestion`.
  Because the seat is interactive, the parent directive drops its operator-ban line, as it does for the interactive Discussion chair.
- **Stencil** (`contracts/stencils/drift/drift-template-verify.md`): the opening line and the Step 2 acceptance paragraph that were fixed to the autonomous mode are now the two markers.
  The autonomous text renders as before.
- **Run counter** (`internal/driftengine/records.go`): `runs.json` gains `operator`.
  `ResetRuns` takes the mode and records it at each fresh spawn, and `OperatorSessionOpen` reads it.
  A resumed live agent keeps the mode it was spawned with.
- **`lym drift accept`** (`internal/driftengine/verbs.go`, `internal/driftcli`): `VerbStatus` carries `OperatorSession`.
  An acceptance made in an operator session is recorded as `by: operator`.
  Under scope full it is ruled, so the row does not raise a second decision halt for the parent over drift the operator already decided.
  An operator session may also re-accept a test the parent withdrew; an autonomous session is still refused.
  The help text of `accept` and `withdraw` says so.

## Design decisions

- **Operator acceptances are ruled.**
  Today, under scope full, an agent's acceptance stays unruled until the run's parent rules on it at a decision halt.
  With the operator deciding in the pane, that halt would ask the parent to approve the operator's own decision again, so the operator's acceptance counts as ruled.
- **The mode lives in the run counter, not in a config read at accept time.**
  `lym drift accept` judges by how the live session was spawned.
  Flipping `verify_interactive` while a session runs therefore changes nothing for that session.
- **The mode and judging text are Go constants**, the same way loomengine's `modeRules` fills the Discussion chair's `mode_rules`.
  The stencil itself stays free of conditionals.
- **The session timeout is unchanged.**
  The verify seat keeps its fixed 120-minute timeout in both modes; `drift.yaml` still has no timeout key.

## Tests

- `config_test`: the key loads.
- `configreg_test`: the key is classified local-open.
- `seats_test`: in each mode, checks `Table.Interactive` and the rendered prompt text.
- `records_test`: the operator mark through `ResetRuns` and `CloseAgentSession`.
- `verbs_test`: operator-session rows, covering partial scope, a ruled full-scope acceptance, re-accepting a withdrawn test, and a mark left over from a closed session.
- `verify_test`: under scope full, an interactive session accepts through the run counter's mark, and its run ends Done with the acceptance ruled.
- `contracts/stencils/markers_test`: pins the two new markers.
