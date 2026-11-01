# A cold, cheap fixer clears a failed writer gate instead of the writer at full context

`cold-gate-fixer` · Task · enhancement, webster, loom, gate · draft

## Seen (orch, 2026-11-01)

A writer that fails its own gate fixes the failures in its own session, at its full context per turn.
Before project `agent-turns`, one Webster fork spent 69 turns at 150k to 450k fixing its card gate.
The Plan writer runs `validate-plan` through its gate at 400k to 670k context in large runs, and the Discussion writer runs `validate-discussion` the same way.
The gate reports name each failing check and its place, so most fixes are mechanical.
R1 (`agent-turns-effect`) measures how much the card gate loop still costs after `gate-failure-report` and `plan-edits-before-gate`.

## Idea (operator, 2026-11-01)

- When a writer's gate fails, Go spawns a cold, cheap fixer with the gate report and the files it names, instead of the writer fixing in its large session.
  The fixer reads the failing source itself, fixes every failure in one go and reruns the gate; when it clears the gate the run goes on, and when it does not the failure goes back to the writer as today.
- Three places: the Webster card gate (as R1's measure may recommend), the Plan writer's plan gate, and the Discussion writer's gate.
- Webster's `lym webster fix-verify` (`webster-prefix-verify-fix-cold.md`) is such a fixer for the plan-level verify gate already, and the pattern to follow.
- Open: which failures a cold fixer may take (format and machine checks) and which need the writer (a check whose fix changes a design choice), and the fixer's model.
- R1's numbers for the card gate, and a count of plan and discussion gate loops in landed runs, decide whether each place is worth it.
