# Cap lyx gate test's slot wait under the prompt-cache lifetime

`lyx gate test` blocked an agent's Bash call for up to the hub's `cli_wait_sec` (template 300 s) while it waited for a free gate slot.
A blocked tool call makes no API request, so a wait past the 5-minute prompt-cache lifetime left the agent on a cold cache, which costs more than the wait.
`lyx shuttle await` already avoids this: `shuttleengine.DefaultAwaitCap` caps one call, the verb answers ready false, and the caller re-runs it.
The gate now follows the same rule with the same cap.

## What changes

- `lyx gate test` waits for a slot for at most `shuttleengine.DefaultAwaitCap`, imported rather than copied.
  When every slot is still held at the cap, it prints an ok envelope with `slot_free: false`, the holders by worktree and site, and a message to run the same command again.
  It exits 75 as before, and runs no go test.
- The gate rule in the shell guard's raw-go steer tells agents to re-run on that answer in a turn that does nothing else, in the words the await help uses.
  The webster implementer and verify-fix stencils describe the new answer's shape.
- `cli_wait_sec` is retired.
  It leaves `Config`, `Limits` and the template, and the gate spec lists it in `RetiredKeys` as `CategoryCrossPair`.
  A hub file that still sets it loads, and `lyx config reconcile` prunes it.
- `cmd/testtiming` takes the shared cap for its slot wait, since `Limits` no longer carries one, so a stuck holder still ends the measurement.
- Docs: `internal/gatecli/doc.go`, `internal/gateslot/doc.go`, the cap's comment in `internal/shuttleengine`, and `docs/overview.md`.

## Design decisions

- Exit 75 stays, now with an ok envelope.
  The code is already distinct from go test's codes and the signal codes, and the stencils already treat it as no failure.
  The ok envelope matches `lyx shuttle await`'s ready false answer.
  A caller that parsed the old `ok: false` busy envelope breaks; no Go caller does.
- `newCommand` takes the slot-wait cap as a parameter.
  `Command` passes the shared cap and the integration helper passes a short one, so the busy-hub test never waits out 250 s and no package variable breaks parallel tests.
- A long test run after the slot is taken is out of scope; the test budget governs it.

## To do by hand

- The hub's `loom.yaml` still sets `cli_wait_sec: 180`; reset it after landing, and `lyx config reconcile` prunes the retired key.
