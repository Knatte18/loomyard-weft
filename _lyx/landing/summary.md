# Cap the gate slot wait at the keep-cache-warm cap, retire cli_wait_sec

An agent running `lyx gate test` blocks in its Bash call while the gate waits for a free slot.
The wait was bounded by the hub's `cli_wait_sec` (template 300 s), so a wait plus a test run could outlast the 5-minute prompt-cache lifetime and leave the agent on a cold cache.
`lyx shuttle await` already keeps one call under that lifetime with `shuttleengine.DefaultAwaitCap`; the gate now follows the same rule with the same constant.

## What changes

- `internal/gatecli` bounds the slot acquire by `shuttleengine.DefaultAwaitCap`.
  `newCommand` takes the wait as a parameter, so `Command` passes the shared cap and the integration helper passes one second.
- When the cap passes with every slot held, the verb prints an ok envelope, `{"ok": true, "slot_free": false, "holders": [...], "message": ...}`, and exits `SlotBusyExit` (75).
  The message tells the caller to run the same command again in a turn that does nothing else, so the prompt cache stays warm between calls, the words the await help uses.
  The verb's help shows the cap and the exit code from the Go values.
- `internal/gateslot` drops `cli_wait_sec` from `Config`, the template and `Limits`, and lists it in the gate spec's `RetiredKeys`.
  Every load drops a key a hub still sets, and the hub-file reconcile prunes it because it is no longer a template key, so the hub's current `cli_wait_sec: 180` keeps loading until reconcile removes it.
- The shell guard's gate rule, which every spawned agent gets in its system prompt, adds the re-run sentence; the claude settings golden carries it.
- The webster implementer and verify-fix stencils describe the busy answer as `slot_free: false` naming the holders instead of a JSON error.
- `cmd/testtiming -resources` waits for its slot for at most the same shared cap, since `Limits` no longer carries a wait.
- Docs: `internal/gatecli/doc.go`, `internal/gateslot/doc.go`, the cap's comment and package doc in `internal/shuttleengine`, and the gate row of `docs/overview.md`.

## Design decisions

- Exit code 75 stays.
  It is already the verb's own code, distinct from go test's codes and the signal codes, and the stencils already count it as no failure.
  Exit 0, as await uses, would let a caller mistake a skipped run for a passing one.
  What changes is the envelope: `ok: true` with `slot_free: false` instead of `ok: false` with an error, so the answer no longer reads as a failure.
- `cli_wait_sec` is retired rather than bounded by the cap: every useful value would be at or below the cap, so the key would be a knob with no reason to turn it.
- `gatecli` imports the constant from `shuttleengine` instead of moving it to a new package: `gatecli` already depends on `shuttleengine` transitively, so the import adds no edge to the build graph.
- A long test run after the slot is taken stays out of scope, as the board entry says; the test budget governs it.

## Testing

- `internal/gatecli` integration: the busy-hub row asserts exit 75, `ok: true`, `slot_free: false`, the re-run message and the holders, and that no go test ran.
- `internal/gateslot`: a new `TestLoadConfig` row loads a gate.yaml that still sets `cli_wait_sec`, and the template and `Limits` rows carry no CLI wait.
- `internal/shuttleengine/claudeengine`: the deny-notice test requires the re-run sentence, and the settings golden carries it.
- Fixtures that wrote `cli_wait_sec` in `hubgeom`, `verifytree`, `websterengine` and `planglyph` tests no longer write it.

## Follow-up

The orch resets the hub's `cli_wait_sec: 180` after landing; `lyx config reconcile` prunes the key from the hub file.
