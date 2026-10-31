# The hibernate request is carried by dobby, not a daemon of its own

`hibernate-in-dobby` · Note · enhancement, dobby

## Seen (operator, 2026-10-31)

`lym hibernate request` runs a daemon of its own that waits for the runs to land and every session to pause before it hibernates the machine.
Dobby already drives every run of its worktree to its end, and `landing-watch` gives dobby and the hibernate daemon one shared test for a run past its Land row.

## Idea

The hibernate request moves into dobby, so no daemon of its own waits on the runs.

## Open

- A request spans every beam, since `--after-all` waits on all runs while a dobby sees only its own: which dobby carries it, or how the dobbies agree.
- A hub pause must leave the dobby that waits outside it.
