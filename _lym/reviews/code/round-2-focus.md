---
round: 2
focus:
  - "internal/reedcli/window.go terminalWindowTitle and internal/reedcli/window_test.go: does any test run the real terminalWindowTitle, covering its base-name fallback for a directory in no worktree and its console-title path for a worktree in a hub fixture, rather than replacing it through the windowTitle seam?"
  - "internal/loomcli/sharedbootstrap.go statusWindowTitle: is the status window's name formed through the shared console title function, or is its omission stated and justified in _lym/landing/summary.md, and do the code, its test and docs/overview.md agree?"
  - "internal/boardengine/handles.go worktreeWindowTitle and the doc comments of IDEWindowTitle and TerminalWindowTitle: do the docs match what an unreadable board yields for a beam or orch worktree, and do handles_test.go rows cover a task under a done beam, a task whose project is missing from the board, and an empty primeName?"
  - "internal/battencli/commanddeps_test.go and internal/battencli/wire.go: do the test name and comments match the console title the commandDeps Title now returns, and does a row fail if a hub shortname or project handle is lost?"
---
