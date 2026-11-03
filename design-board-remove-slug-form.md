# Way-forward texts name lym board remove <slug>, which the verb refuses

`board-remove-slug-form` · Task · bug, board · running · Worktree-Create

Several way-forward texts tell the reader to run `lym board remove <slug>`, a form the verb refuses.

## Seen

On the `cs` hub (2026-11-03), removing the adopt fix task `adopt-fix-root-t1` as the fix loop's notice names it:

```
$ lym board remove adopt-fix-root-t1
{"error":"invalid json: invalid character 'a' looking for beginning of value","ok":false}
```

`lym board remove '{"slug":"adopt-fix-root-t1"}'` worked.
`lym board remove --help` takes a JSON object with `slug`, `id` or `handle`, or a bare handle such as `T4`, never a bare slug.

The bare-slug form is named in:

- `internal/adoptcli/fixloop.go` (the fix loop's drop line) and `internal/adoptcli/green.go` (a failed fix-task removal)
- `internal/fronttask/fronttask.go` (`RemoveError.HandRemoval` and the one-open-front-task refusal)
- `internal/boardengine/front.go` (the fixed-front-task refusal)
- `internal/adoptcli/doc.go`

## Expected

Every way forward names a command the verb accepts: either `lym board remove` takes a bare slug as it takes a bare handle, or each text names `lym board remove '{"slug":"<slug>"}'`.
A test pins that each way-forward command these packages render parses.
