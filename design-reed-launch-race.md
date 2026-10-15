# reed: parallel window launches corrupt the launch line

`reed-launch-race` · Note · bug

Imported from [issue #627](https://github.com/Knatte18/loomyard/issues/627).

Seen (2026-10-15): several lyx shuttle run windows launched at once in the prime's tmux session. Window loomyard:5 received "5. '<prime>/.lyx/reed/launch/e97d8002568306288d817e5f23215a96.sh'" and window loomyard:6 received "0. '<prime>/.lyx/reed/launch/10654afc6d30051dd4405708cafd6503.sh'"; bash answered '5.: command not found' / '0.: command not found' and Claude never started, so the shuttle run hung. Workaround: launch one window at a time. Expected: concurrent launches each get their own intact launch line.