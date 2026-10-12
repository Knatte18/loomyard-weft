# A run window closes itself when its run ends

`run-window-close` · Note · enhancement, batten

## Seen (2026-10-12)

`lyx batten run <slug> --window` opens one Konsole window per run that runs `lyx reed attach` in the pair (`internal/termwindow/termwindow.go`).
`attachLine` chains `exec bash` after the attach on purpose, so the window stays open as a shell once the attach returns.
At Worktree-Teardown the pair and its reed session go away, the attach returns, and the operator is left with an open shell in a deleted directory, one window per finished run.

## Want (operator, 2026-10-12)

When batten is done with a run, the same window closes.

## Open

- Close only on a clean end (the run landed or was torn down), and keep the shell when the attach fails or returns early, so an error stays readable.
- Whether a detach by the operator, the attach returning while the run lives, also closes the window.
- The WSL launcher (Windows Terminal) should behave the same.
