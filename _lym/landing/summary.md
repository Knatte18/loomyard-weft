# Way-forward texts name a `lym board remove` command the verb accepts

Several way-forward texts told the reader to run `lym board remove <slug>`.
The verb takes a JSON object with `slug`, `id` or `handle`, or a bare handle such as `T4`, and refuses a bare slug with `invalid json`.
Every way forward in `adoptcli`, `fronttask` and `boardengine` now names `lym board remove '{"slug":"<slug>"}'`, which the verb runs.

## What changes

- `boardengine.RemoveCommand(slug)` renders the command line, with the JSON payload built by `json.Marshal` and single-quoted for a shell.
- These texts render their removal command through it:
  - `adoptcli` (`judgeOwnFix`, the fix loop's drop line) and the package doc in `internal/adoptcli/doc.go`.
  - `fronttask` (`RemoveError.HandRemoval` and the one-open-front-task refusal in `File`).
  - `boardengine` (the fixed-front-task refusal in `front.go`).
- `removeFixTask` in `internal/adoptcli/green.go` no longer carries a fallback line for an error that is not a `*fronttask.RemoveError`, which named the bare-slug form.
  `fronttask.Remove` returns only that type, so the line was unreachable.

## Design decisions

- The slug stays out of the verb's input grammar: the verb is unchanged, and the texts name the form it already accepts.
  Accepting a bare slug would clash with the bare-handle form, which a slug such as `t4` could shadow.
- One renderer in `boardengine` keeps every text on the same command, and the test below runs that renderer's output.

## Tests

- `stepRemoveCommandParses` in `internal/boardcli/verbs_test.go` takes the command `RemoveCommand` renders for several slugs, strips the shell quotes, and runs the payload through the remove verb.
  It expects the entry gone afterwards.
- `stepFrontTask` pins that the fixed-front-task refusal names the same command the verb then runs.
- The `adoptcli` and `fronttask` tests expect the JSON-payload form in the notices they render.

## To check by hand

- `removeFixTask` in `internal/adoptcli/green.go` asserts `err.(*fronttask.RemoveError)` without the comma-ok form, so a future error of another type from `fronttask.Remove` would panic.
  Check that `fronttask.Remove` returns only `*fronttask.RemoveError`, and that `errors` is still used in the file.
- The code review's finding `remove-command-parse-test-missing` (MEDIUM) was answered by `stepRemoveCommandParses` and no fresh review has seen the result.
  Check that the test covers every command these packages render, not only the ones it builds through `RemoveCommand`.
- Run the real flow once: trigger the fix loop's notice on a hub, and run the removal command it prints.
