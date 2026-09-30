# Webster round 1: smoke tier results depend on the burler's own $TMUX

The burler session runs inside a reed tmux pane, so `$TMUX` is set in every shell it opens.
`TestSmokeBootstrap_MalformedStatusProceedsToHandoverAndLogsWhy` and `TestSmokeBootstrap_DiedDriverProceedsToHandoverAndLogsWhy` then fail: `lyx loom start` takes the inside-another-tmux branch and returns a hint envelope instead of reaching the attach the tests expect.
With `TMUX` and `TMUX_PANE` unset, both pass.

It cost a detour to rule out the diff as the cause.
The smoke helpers (`runLoomCLINoFatal`) inherit the parent environment, so every agent that runs the smoke tier from a reed pane will hit this unless the helper strips `TMUX` or the handover tests set it explicitly.
