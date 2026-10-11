---
verdict: APPROVED
---

No defects found.
Review range: `git diff $(git merge-base main HEAD)..HEAD` (`product.parent` is `main`), seven files.

## Focus questions settled

- **`OpenWindow` argv** (`internal/reedengine/window.go:72-74`): the chained `set-option -w -t '=<session>:=<name>' @lyx_hidden 1` uses the same exact-name `namedTarget` as the existing `remain-on-exit off`, so it lands on the new window and never on the strands' window.
  It follows `new-window` in one multiplexer invocation, so it cannot run when `new-window` fails.
  `remain-on-exit off` is untouched, and the read-back guard still covers the shared-name race.
  `window_test.go` pins the argv and `window_integration_test.go` reads `@lyx_hidden` as `1` on the opened window.
- **`statusFormatButtons`** (`internal/reedengine/bar.go:38`): only the window-button branch gained the `&&` with `!=:@lyx_hidden,1`.
  The `@lyx_strands` branch (strand buttons), VIEW and the session line are unchanged.
  The `session_windows > 1` edge the summary names is stated accurately.
- **Orch stencil**: the new wording at `contracts/stencils/orch/orch-template-role.md:21` licenses only watching via `lyx batten status <slug>`, which the same stencil already lists at line 29.
  No other doc describing the window list or status line 1 is left stale: `docs/overview.md:513` and the `--window` help in `internal/battencli/cli.go` say only that the window lives in the reed session.
  `doc.go` was updated in the same diff.
  `@lyx_hidden` is display only and no Go code reads it, so no new `PATTERN` entry is needed.
- **Repository rules**: docs land in the same diff, no new top-level test function is added so `cmd/lyx/testdata/test-budget.yaml` needs no edit, and the changed `.md` keeps its line shape.
- **Tests**: each of the three tests extends the existing test for its behavior (the argv table, the `OpenWindow` real-tmux test, the bar fixture test), so none duplicates another and no added behavior is uncovered.
