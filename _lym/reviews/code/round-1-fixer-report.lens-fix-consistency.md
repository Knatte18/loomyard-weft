Task: check the change set (F1 test step, F2 fallback removal) for consistency with its surroundings.

No findings.

- The only wording the removed fallback carried was "was not removed (%v)". A grep of `internal` finds no other copy of that text, so no twin was left behind.
- Every `lym board remove` mention in `internal` that isn't followed by `'` is absent from the grep, so no bare-slug way forward remains.
- `fronttask.Remove` returns only `*RemoveError` or nil (`fronttask.go:222-247`). The `err.(*fronttask.RemoveError)` assertion in `removeFixTask` therefore agrees with its documented contract at `fronttask.go:221`.
- The `removeFixTask` doc comment ("the line carries the failing step's error with the command that finishes it by hand") still matches the single remaining branch.
- The `errors` and `boardengine` imports in `green.go` are still used, at lines 6, 257 and 209.
- The new `RemoveCommandParses` step is registered in the `cli_test.go` step list and defined in `verbs_test.go`. It follows the same seed, run and get pattern as the `FrontTask` step beside it.

Tool calls:
1. `git show --stat` of both fix commits, to confirm which files changed. My context showed the edits but not the commits.
2. `grep` for the removed fallback wording and for non-quoted `lym board remove` forms, to look for twins left unchanged.
