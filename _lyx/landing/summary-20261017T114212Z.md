# Cap lyx gate test's slot wait at the keep-cache-warm cap

`lyx gate test` blocked an agent's Bash call while it waited for a free gate slot, bounded only by the hub's `cli_wait_sec` (template 300 s).
A blocked tool call makes no API request, so a wait plus test run past the 5-minute prompt-cache lifetime left the agent on a cold cache, which costs far more than the wait.
`lyx shuttle await` already avoids this: `shuttleengine.DefaultAwaitCap` caps one call under the cache lifetime, and the caller re-runs it.
The gate now follows the same rule.

## What changes

- `lyx gate test` waits for a slot at most `shuttleengine.DefaultAwaitCap`.
  `gatecli.Command` passes the shared constant, not a copy; the cap's doc comment now covers every keep-cache-warm wait.
- When every slot is still held at the cap, the verb prints an ok envelope with `slot_free: false`, the holders by worktree and site, and a message to run the same command again.
  It exits `SlotBusyExit` (75) as before and runs no `go test`.
  The envelope is no longer `ok: false`, so it reads like `lyx shuttle await`'s ready-false answer rather than an error.
- `cli_wait_sec` is retired: it leaves `gateslot.Config`, `Limits` and the template, and the gate spec lists it in `RetiredKeys` as `CategoryCrossPair`.
  A hub file that still sets it keeps loading, and `lyx config reconcile` prunes the key.
- The gate rule in `internal/shellguard` tells agents to re-run on a no-slot-free answer in a turn that does nothing else, in the words the await help uses.
  The webster implementer and verify-fix stencils describe the new answer shape.
- `cmd/testtiming` takes the shared cap for its slot wait, since `Limits` no longer carries one; a stuck holder still ends the measurement.
- Docs: `internal/gatecli/doc.go`, `internal/gateslot/doc.go`, `internal/shuttleengine/doc.go` and `docs/overview.md`.

## Design decisions

- Exit code 75 is kept rather than a new number: it is already the verb's own code, distinct from `go test`'s and the signal codes, and callers already treat it as no failure.
- A bound the operator could raise past the cap would break the rule, so the key is retired rather than clamped.
- `newCommand` takes the slot-wait cap as a parameter, so the busy-hub integration test passes a short one instead of waiting out 250 s; a package variable would break parallel tests.
- A long test run after the slot is taken stays out of scope; the test budget governs it.

## Compatibility

A caller that parsed the old busy answer's `ok: false` error envelope breaks.
No Go caller reads it, and the stencils that mention it change here.

## To check by hand

- The hub currently runs `cli_wait_sec: 180`.
  After landing, run `lyx config reconcile` on the hub (or reset the value) so the retired key is pruned.
- The `settings-golden.json` fixture under `internal/shuttleengine/claudeengine/testdata` changed with the gate rule text; confirm the diff is only that wording.
