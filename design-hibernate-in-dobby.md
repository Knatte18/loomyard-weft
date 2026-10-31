# The hibernate request is carried by dobby, not a daemon of its own

`hibernate-in-dobby` · Note · enhancement, dobby

## Seen (operator, 2026-10-31)

`lym hibernate request` runs a daemon of its own that waits for the runs to land and every session to pause before it hibernates the machine.
Dobby already drives every run of its worktree to its end, and `battenrecipe.PastLand` is the one past-Land test that dobby and the hibernate daemon share (built by `landing-watch`).

## Idea

The hibernate request moves into dobby, so no daemon of its own waits on the runs.

## Open

- The shared test is built; moving the request is the open part.
- A request spans every beam, since `--after-all` waits on all runs while a dobby sees only its own: which dobby carries it, or how the dobbies agree.
- A hub pause must leave the dobby that waits outside it.
