# Webster verify gate escalates to a cold Opus fixer

A plan-level verify failure in Webster that the fixer fork does not clear now goes to a cold fixer on the recovery model (`opus[high]` by default) before the run halts.
Before, every attempt went to a fork on Merriam's own model (`sonnet[medium]`), and nothing after the halt fixes code.

## What changes and why

1. **The gate routes each failure to a fixer.**
   `NewVerifyGate`'s closure records the routed fixer as `fixer: fork` or `fixer: cold` in `verify-gate.yaml`.
   A failure goes cold when it is the last failure the attempt budget re-prompts, when the previous fixer added no commit above the pre-fix head, and for every failure after one that went cold.
   The findings and the re-prompt line open with the way forward to that fixer: the fork step with the prompt file, or a backgrounded `lyx webster fix-verify`.
   `VerifyGateNotes` carries the routing to the re-prompt renderer.
2. **`lyx webster fix-verify` runs the cold fixer.**
   The new role verb spawns a strand with role `verify-fixer` at `RoleRecovery`'s model choice, and blocks until it ends or `recovery_timeout_min` passes, as `recover-batch` does.
   The fixer's final action is a reply file in Webster's scratch dir (`.lyx/webster/verify-fix-reply.md`), which is the strand's shuttle output file.
   The verb returns `done` with the reply, or `dead` with `asking`, `timeout` or `died` and a non-zero exit.
   Every terminal result stops the strand, and only `done` removes the run dir.
   A call that finds a fixer still running attaches to it.
   The verb refuses before spawning when no verify-gate report is on disk, when the recovery role's engine cannot audit forks, or when the gate config has no `round` list.
   It also refuses under a pause that covers the pair, through the pause refusal `begin-batch` and `recover-batch` use, now generalized as `refuseCovered`.
3. **The cold fixer is a run session like a recovery.**
   `State.VerifyFixers` records each spawn: strand, session, engine, spawn time, events and terminal status.
   Fixer sessions join the run's write evidence, run entry stops a fixer that is not terminal, and `reset --to start` stops every recorded fixer.
4. **One fixing body, two prefixes.**
   `webster-body-verify-fix` now holds only the findings and the fixing rules.
   The new `webster-prefix-verify-fix-fork` carries the fork's in-session framing and its reply, and `webster-prefix-verify-fix-cold` carries the cold framing, the parent, edit, PATTERN and friction directives, the orientation steps and the reply file.
   The cold prefix is the `verify-fixer` role's opening stencil in the registry, so the parent and edit directive tests cover it.
5. **Merriam's gate-failure section** runs `fix-verify` backgrounded when the way forward names it, and on its completion, whatever the exit, rewrites its contract files and ends its turn so the gate verifies again.
   Master's spawn now declares `lyx webster fix-verify` as an awaited shell prefix beside `recover-batch`.
6. **The `webster.yaml` template's `verify_gate_attempts` drops from 3 to 2,** so an untuned hub runs one fork attempt, then one cold attempt.

Docs: `internal/websterengine/doc.go` gains "The cold verify-gate fixer" section, and `internal/webstercli/doc.go`, `contracts/specs/webster-spec.md`, `contracts/specs/llm-model-spec.md` and `docs/overview.md` are updated.

## Design decisions to know

- The escalation sits in the gate, which already owns the attempt count and the fix-commit range.
  The gate only names the fixer, and Merriam starts it: the gate never spawns, since a spawn there would block shuttle's wait loop for up to the recovery timeout.
- The cold fixer reuses the `recovery` model key and `recovery_timeout_min`, so `webster.yaml` gains no new key.
- The reply file sits in `.lyx`, never under `_lyx`, so the shared rule that a fixer never touches `_lyx` holds for both fixers.
- Bound: `fix-verify` spawns whenever a verify-gate report exists, also after a failure the gate routed to the fork, and counts no spawn.
  What caps the cold fixers in a run is the number of gate messages, which the attempt budget limits, and the Master template's rule to run the verb only when a message names it.
- `state.json` gains an optional `verifyFixers` key, and a state without it loads as before.

## To check by hand

- A re-prompt shuttle could not deliver is evaluated again, which counts one more failure in the closure, so routing can go cold one attempt early; the cap still bounds the run.
- A hub that tuned `verify_gate_attempts` keeps its value and gets the cold fixer only on its last attempt, or after a fixer that committed nothing.
- No live run has exercised `fix-verify` yet: the tests drive it over the shuttle fakes.
  The first real gate failure under the new routing is worth watching for the cold prompt's orientation and the reply file landing.
