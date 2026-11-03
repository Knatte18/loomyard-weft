# Way-forward texts name `lym board remove` with a JSON slug payload

Several refusals and notices told the reader to run `lym board remove <slug>`.
The verb takes a JSON payload with `slug`, `id` or `handle`, or a bare handle such as `T4`, and refuses a bare slug as invalid JSON, so the way forward they named failed.

## What changed

- `boardengine.RemoveCommand(slug)` renders `lym board remove '{"slug":"<slug>"}'`, its payload built with `json.Marshal`.
- Every way forward that named the bare-slug form now names `RemoveCommand`'s output:
  - `boardengine`'s refusal of a change to a front task's project, kind or slug;
  - `fronttask`'s `RemoveError.HandRemoval` and its one-open-front-task refusal in `File`;
  - `adoptcli`'s fix-loop drop line in `judgeOwnFix` and its failed fix-task removal in `green.go`.
- `internal/adoptcli/doc.go` names the JSON form.

## Design decisions

- The texts change rather than the verb.
  `boardengine.ParseHandle` matches a handle's letter case-insensitively, so a bare argument such as `t4` already reads as a handle; teaching `resolveLookup` to take a bare slug too would make a slug of that shape ambiguous.
- One renderer in `boardengine`, the lowest package the three share, keeps the command's form in one place.

## Tests

- `boardcli`'s `stepFrontTask` asserts that the fixed-front-task refusal names `lym board remove '{"slug":"fix"}'`, then runs the remove verb with that same payload and checks the entry is gone, pinning that the command the refusal renders parses.
- The `fronttask` and `adoptcli` tests that asserted the bare-slug form now assert the JSON form in each way forward they cover.
