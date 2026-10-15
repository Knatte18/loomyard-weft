# reed: intact launch lines under parallel launches, --focus=false never takes focus

`reed-launch-fixes` · Note · bug, reed, shuttle

Imported from [issue #627](https://github.com/Knatte18/loomyard/issues/627) and [issue #628](https://github.com/Knatte18/loomyard/issues/628).

## 1. Parallel launches corrupt the launch line (#627)

Seen (2026-10-15): several `lyx shuttle run` windows launched at once in the prime's tmux session.
Window `loomyard:5` received `5. '<prime>/.lyx/reed/launch/e97d8002568306288d817e5f23215a96.sh'` and window `loomyard:6` received `0. '<prime>/.lyx/reed/launch/10654afc6d30051dd4405708cafd6503.sh'`; bash answered `5.: command not found` / `0.: command not found` and Claude never started, so the shuttle run hung.
Workaround: launch one window at a time.
Expected: concurrent launches each get their own intact launch line.

## 2. `--focus=false` windows still take focus (#628)

Seen (2026-10-15): `lyx shuttle run --focus=false` from the prime opened each agent window in the prime's tmux session and switched focus to it, pulling the operator away from the orch pane.
Expected: `--focus=false` never switches the active window.

Also seen, in scope only if the fix needs it: there is no flag to open the window outside the prime's session, and `--anchor hidden` died before Claude started ("provider never became ready").
