# verify-nested-module misses commands after || and |

`verify-module-or-pipe` · Note · bug, loom

## Seen

The PR-Gate review of `plan-gate-checks` (PR #475) found that `crossModuleCommands` in `internal/planparser/verifymodule.go` splits a verify chain only at `&&`, `;` and newlines, so a `go` command after `||` or a pipe is read as part of the previous segment and its module is never checked.

## Idea

Split at `||` and `|` too.
