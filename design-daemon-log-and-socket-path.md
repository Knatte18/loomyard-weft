# daemon idle Debug spawn lines reach no log sink; tmux socket path can exceed 108 bytes

`daemon-log-and-socket-path` · Note · bug, reed

Imported from [issue #495](https://github.com/Knatte18/loomyard/issues/495).

Two smaller frictions while measuring the daemon's idle behavior:

1. The daemon's idle Debug spawn lines never reach any sink, so spawn counts could not be read from lyx logs.
   The agent counted spawns through a tmux wrapper instead.
   Suggested fix: route those Debug lines to the daemon log at a verbosity the operator can enable.
2. A tmux socket path under a deeply nested scratch directory exceeded the 108-byte socket limit.
   `TMUX_TMPDIR` had to be set to a short path by hand.
   Suggested fix: have lyx detect an over-long socket path and fail with a message naming `TMUX_TMPDIR`, or choose a short default location.

Provenance: task `daemon-wakeups`; note `09-orch-held-injection-log.md`.
