# Board promote sets depends_on in the write that releases a Sketch

`lym board promote` of a Sketch gives the task recipe `raddle` and releases it in the same write, so that its parent's dobby starts the raddle run.
An orch that promoted a Sketch and then ran `lym board set-deps` had a window between the two writes.
In that window the dobby saw a released task with no dependencies and started it.
The dobby's own check was never the fault: it holds a released task while any `depends_on` entry is not done or gone, whatever the recipe.
The task simply had no `depends_on` yet when the dobby read it.

## Change

- `lym board promote` takes an optional `depends_on` in its payload, such as `lym board promote '{"handle":"S3","depends_on":["other-task"]}'`.
  The list is validated and written in the same write that makes the task and, for a Sketch, releases it.
  The dobby therefore never reads the released task without its dependencies.
- On an entry that is already a task, promote still writes nothing.
  A `depends_on` that differs from the task's own is refused, naming `lym board set-deps` as the way forward.
  A retried promote with the same list still succeeds.
- Like `set-deps`, promote now adds the sibling warning to its envelope's `notice` when the new dependencies make a sibling.
- `Board.PromoteTo` takes the dependency list and returns the siblings the write created.
  `set-deps` and promote share one parser for the `depends_on` value.
- The promote help, `internal/boardcli/doc.go` and `internal/boardengine/doc.go` say to give a Sketch's dependencies in the promote itself.

## Design decisions

- The promotion still releases a Sketch in the same write.
  The documented flow, in which a promoted Sketch starts its raddle run with no separate release, stays the same.
  The fix closes the window by letting the dependencies arrive in that same write.
  A Sketch promotion that leaves the task as a draft would have changed that flow for every caller.
- The dobby needed no change, since its start condition already treats every recipe the same way.

## Tests

- `TestDiscussedPromotion` has two new rows.
  One pins a Sketch promoted with `depends_on` as a released raddle task that carries them.
  The other pins the refusal for an entry that is already a task.
- The board CLI steps cover a promote with `depends_on`, a non-array `depends_on`, an unknown dependency, and the help text and payload declaration.
