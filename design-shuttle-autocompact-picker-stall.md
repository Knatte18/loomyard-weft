# Claude Code autocompact picker in a shuttle session stalls a gate re-prompt forever

`shuttle-autocompact-picker-stall` · Note · bug, shuttle, loom

## Seen (2026-11-02)

In run `finalize-fix-outside-lock`, the Code-Burler-Fix fixer finished its turn.
Claude Code then showed its "We recommend compacting earlier / autocompact at 400k?" picker in the shuttle session, where it replaced the input box.
The Code-Burler-Fix verify gate found one fixed-column comment wrap and tried to re-prompt the fixer.
The trace logged "re-prompt not delivered, retrying on a later tick without spending an attempt … no input box is on screen; a menu or picker" once a minute for over 30 minutes.
The re-prompt spends no attempt and never escalates, so the run waits forever; only dobby's quiet notice surfaced it.

## Workaround

A single Escape key in the fixer's pane closes the picker ("esc not now"); the gate's next tick then delivers the re-prompt.

## Expected

- lym keeps Claude Code's autocompact picker out of shuttle sessions, for example by setting the autocompact choice in the session's settings at launch.
- A re-prompt that cannot be delivered because a picker holds the pane escalates after a bounded number of ticks, or dismisses a known picker itself, instead of retrying without end.
