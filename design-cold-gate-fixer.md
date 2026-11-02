# One cold, cheap fixer clears a failed writer gate instead of the writer at full context

`cold-gate-fixer` · Task · enhancement, webster, loom, darn, gate · done

## Seen (orch, 2026-11-01)

A writer that fails its own gate fixes the failures in its own session, at its full context per turn.
Before project `agent-turns`, one Webster fork spent 69 turns at 150k to 450k fixing its card gate.
The Plan writer runs `validate-plan` through its gate at 400k to 670k context in large runs, and the Discussion writer runs `validate-discussion` the same way.
The darn writer of `gate-one-binary-per-package` fixed its own `darn-write` gate failures at about 274k context, and darn is the recipe that runs most.
The gate reports name each failing check and its place, so most fixes are mechanical.

## Design (operator, 2026-11-01)

- One shared cold gate fixer, not four: when a writer's gate fails, Go spawns a fresh, cheap fixer with the gate report and the files it names, instead of the writer fixing in its large session.
  The fixer reads the failing source itself, fixes every failure in one go and reruns the gate, the failed packages first and then the whole gate; when it clears the gate the run goes on, and when it does not the failure goes back to the writer, or up the recovery ladder, as today.
- Webster's `lym webster fix-verify` (`webster-prefix-verify-fix-cold.md`), the cold fixer of the plan-level verify gate, is generalised into the shared one rather than built beside.
- Four callers:
  - the Darn-Write gate (`darn-write`);
  - the Plan writer's plan gate;
  - the Discussion writer's gate;
  - the Webster card gate, where Go spawns the fixer in the fork's worktree with the gate report, the card and the card's diff.
- Which failures the fixer takes: the failures a gate reports as machine checks go to it; a failure whose fix changes a design choice goes back to the writer. The Discussion settles the line from what the gate reports contain.
- The fixer's model and effort are config, one key per caller, with a cheap default; the Discussion picks the default.
- R1 (`agent-turns-effect`) measures the card gate loop after this lands.
