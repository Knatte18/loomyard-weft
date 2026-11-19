```yaml
lens: fix-code-correctness
outcome: reported
```

no findings

I ran the changed paths step by step in my head and checked that each added test would fail with its defect in place.

- **F1 (`watchBoard`)**: the first event reads at once because `lastRead` is zero. A later event waits out `watchEventFloor`, then `drainEvents` empties the queue. A closed channel sets `events = nil` and takes the timer path.
  - The burst test would fail without the floor: it asserts one read while the floor wait is registered.
  - The back-off test traces to `[1,2,4,8,8,1]s`, which matches the assertion.
- **F2 (server and page)**:
  - `?after=` on a board with no `board.json` now waits for the bound and answers unchanged, and the new `TestServer` row fails under the old `after == ""` rule.
  - The page sends no `after` only on its first poll, and a retry after a failure keeps the `null`.
  - The server always returns `revision` on an ok answer, so `?after=undefined` cannot happen.
- **F5 (idle stamp)**: `NoToken`, `WrongToken` and `WriteWithNoToken` would each move `lastAPIRequest` off `testClock` if the stamp ran before the token check. The row order keeps those checks ahead of the authorized rows that do stamp.
- **F6 (create with revision)**: the page's `send` omits `revision` when it is `''`, and every create passes `''`, so the page never sends a revision on a create. The new `RefusedWritesChangeNothing` row would fail without the refusal.
- **F7 (project slug)**: the new row would fail without the check, because the error would then be a `StaleEntryError`. The edit path with a non-empty revision is unchanged.
- **F4 (`serveOptions`)**: the callers still compile, since `serveCommand` now passes `config()` and `resolved.HubPath` to it.
  - `BoardCarriesNoStrandName` would fail if the strand name were kept, because a run-agent name refuses a new task.
  - `TestProductionOpenDeps` fails if the cap is set for an unnamed caller. It cannot tell a wrong strand argument from a right one, since both give the fallback cap.
- **F8, F10, F11, F12**: the new assertions and marker comments are consistent with the code they pin.

I made no further tool calls beyond two `grep`/`sed` reads of `internal/boardengine/board.html`: the `send` function's `revision` handling and the create callers and stale-refusal branch (lines 830–875). My context held only the `poll` span of that file.
