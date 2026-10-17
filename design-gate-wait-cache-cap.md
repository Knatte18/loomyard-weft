# lyx gate test slot wait capped under the prompt-cache lifetime, like shuttle await

`gate-wait-cache-cap` · Task · enhancement, config, shuttle · running · Finalize

## Seen (2026-10-17)

An agent that runs `lyx gate test` blocks in its Bash call while the gate waits for a free slot (`internal/gateslot`, `internal/gatecli`).
The wait is bounded by the hub's `cli_wait_sec` (template 300 s), after which the verb exits 75 naming the holders.
A blocked tool call makes no API request, so a wait plus test run past the 5-minute prompt-cache lifetime leaves the agent on a cold cache, which re-writes its whole context and costs much more than the wait.
`lyx shuttle await` already solves this: `shuttleengine.DefaultAwaitCap` (250 s) caps one call under the cache lifetime, the verb answers ready false, and the caller re-runs it in a turn that does nothing else.
The gate keeps its own, larger bound for the same rule.

## Operator decision 2026-10-17

- `lyx gate test`'s slot wait follows the keep-cache-warm rule of the wait verbs, with the same cap, taken from the one shared definition rather than a copy.
- When the cap passes without a free slot, the verb answers that no slot is free yet and the caller is to run it again, with an exit code of its own that is not a failure, naming the holders as today.
- The gate rule agents get in their prompts says to re-run on that answer, in the same words the await help uses.
- `cli_wait_sec` is retired or bounded by the shared cap; `lyx config reconcile` handles a hub that still sets it.
- A long test run after the slot is taken is out of scope; the test budget governs it.
- Until this lands the hub runs `cli_wait_sec: 180`; the orch resets the hub value after landing.
