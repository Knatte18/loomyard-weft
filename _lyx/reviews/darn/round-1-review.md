---
verdict: APPROVED
---

The diff makes the change the board entry asks for: `OpenWindow` reads `#{window_id} #{remain-on-exit}` back and counts a failed read, or a reply whose id is not the new window's, as the window being gone.
It is confined to `internal/reedengine`, which is within the entry's scope.

## Checked

- Review range: `product.parent` is `main`, merge base `a1f435dc`, four files changed.
- tmux 3.4 on this machine, run on a private socket with the session's `remain-on-exit` on: `display-message -p -t @99 '#{window_id} #{remain-on-exit}'` exits 0 and prints ` on`.
  This bears out the change description's and `doc.go`'s claim about tmux's answer for a vanished target.
- `window.go:107-121`, one input per branch:
  - failed read: counted as gone;
  - ` on\n` (empty id): counted as gone, nothing killed;
  - `@1 on\n` (foreign id): counted as gone;
  - `@7 on\n`: kills by id and wraps `ErrWindowReadBack` with `read "on"`;
  - `@7 off\r\n`: opened window returned, since `TrimRight` strips the `\r\n`;
  - `@7\n`: takes the existing `value != "off"` mismatch branch, as the old empty read did.
- Affected set: every fake answer scripted by format (`window_test.go`, `windoworder_test.go`) is updated to `windowReadBackFormat`; the real-tmux tests assert results, not the format; `battencli` uses only `errors.Is` on the sentinel; `doc.go` and the `OpenWindow` doc comment stay true.
- Repository rules: the new behavior is a row in `TestOpenWindow`, not a new test; the row is fake-driven with no clock; docs land in the same diff; the comments follow the Go comment conventions.
- The change description's claims, including that the tmux tier was not run in this session, match the diff.
- Focus questions settled: a reply from the new window with a value other than `off` still kills it and wraps `ErrWindowReadBack`; the split on the first space holds for an empty id, no value and a trailing newline; the id-mismatch branch is covered by the new global-scope row and the genuine mismatch by the `@7 on` row, and no existing test covered the id-mismatch branch.

## Rejected

- Fork code-tests F1, LOW: no row for a reply carrying the new window's id and no value (`@7\n`).
  That input takes the `value != "off"` branch the `@7 on\n` row already pins, so a second row would duplicate its coverage under `PATTERN-test-economy`.
