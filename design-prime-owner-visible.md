# The README and lyx board list show which machine owns the prime

`prime-owner-visible` · Task · enhancement, board · done

## Seen

The machine that owns the hub's prime is recorded only as `prime_owner` at the top of `board.json` on Bolt (`internal/boardengine/store.go`).
The rendered README shows each task's owner machine, but not the prime's, and no `lyx board` verb prints it.
On 2026-10-23 the operator adopted two tasks from Neve on OSL-1046 and learnt only from `lyx adopt`'s warning that Neve still owned the prime, so the runs would halt at Finalize.

## Idea

The README states the prime's owner at its top, and `lyx board list` (both JSON and `--text`) prints it, so the owner of the prime is visible wherever task owners are.
