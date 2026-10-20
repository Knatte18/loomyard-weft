# tender pause does not hold a verify gate's re-prompt, and creel interrupt is refused under the pause

`tender-pause-holds-reprompt` · Note · bug, creel, shed

Imported from [issue #762](https://github.com/Knatte18/loomyard/issues/762).

## Seen (2026-10-20, OSL-1046, darn run arbiter-verdict-quoting)

1. Darn-Write's verify gate was looping on a tmux test the writer cannot fix, so the orch ran `lyx tender pause arbiter-verdict-quoting` (the darn session was idle, listed `already_idle`) and `lyx loom pause --after Darn-Write`.
2. The verify process kept running (attempt 2, then a new attempt), since a pause holds agents and network work but not a running gate command. On the operator's order the orch killed the verify's process group.
3. The gate then re-prompted the darn session with the failure, and the session started a new turn although `lyx tender status` listed it `paused: true` (state `busy`, cause `turn`).
4. `lyx creel interrupt ly:arbiter-verdict-quoting:darn` was refused: "a pause covers ...; nothing was interrupted, lift the pause first".
5. Workaround: running `lyx tender pause arbiter-verdict-quoting` a second time printed `already_paused: true` and `interrupted: [ly:arbiter-verdict-quoting:darn]`, and the session went idle.

## Expected

- A paused scope receives no new prompt from a gate, a driver or any other lyx program: a re-prompt waits for the resume, like other held work.
- A pause also stops, or holds at its next boundary, a verify gate command running in that scope, so the operator does not have to kill processes by hand.
- `creel interrupt` of a session the pause covers interrupts it, since stopping work is what the pause wants.