# reed: shuttle run --focus=false windows still take focus

`reed-focus-false` · Note · bug

Imported from [issue #628](https://github.com/Knatte18/loomyard/issues/628).

Seen (2026-10-15): lyx shuttle run --focus=false from the prime opened each agent window in the prime's tmux session and switched focus to it, pulling the operator away from the orch pane. There is also no flag to open the window outside the prime's session, and --anchor hidden died before Claude started ('provider never became ready'). Expected: --focus=false never switches the active window.