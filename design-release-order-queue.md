# Released tasks start in a queue order the board shows and the operator can change

`release-order-queue` · Note · feature, board, dobby

## Seen (operator, 2026-11-28)

When a run place frees, dobby walks the board's tasks in their stored order and starts the first released one nothing holds (`startReady` in `internal/dobbyengine/starts.go`).
When a task was released plays no part, and the board shows no order: nobody can see which task starts next.
The only way ahead of the others is a front task.

## Idea (operator, 2026-11-28)

- Released tasks form a queue ordered by release time; dobby starts the oldest ready one first.
- The board shows each released task's place in the queue.
- The operator, or the orch on the operator's word, can move a task in the queue.
- A held task keeps its place: when its dependencies are done it is the next to start once a run place frees, never sent to the back.
