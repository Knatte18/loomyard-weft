# Card 11 — benchmark-rerun

**Prosa:**
- `docs/benchmarks/daemon-idle-cpu.md`

**Intent:**
Decision D8: re-run the report's recorded procedure on this task's binary and record what the children are.
This is manual work in a scratch hub, never a test, and it changes no code.

Build this branch with `./deploy-dev`, so the binary under test is `.dev-bin`'s, and run the report's procedure as written: a scratch hub built from local bare repositories in a temporary directory, three idle strand-less worktrees, the spawned daemon ended, the daemon started in the foreground in the shell's background with `LYX_LOG_LEVEL=debug`, the `/proc/<pid>/stat` fields read at three seconds and at six hundred, then the three-hundred-second shim run with `--tmux` pointing at a shell shim that appends the first non-flag argument of every invocation to a file.
Keep the earlier runs' shape, three idle strand-less sessions, so the rows compare.
With card 4 the daemon's watchers spawn through `--tmux`, so the shim now counts every subcommand, and with cards 1 and 3 the Debug spawn lines reach the durable log in the hub logs directory at `LYX_LOG_LEVEL=debug`, which is a second count to cross-check the shim against.

Append to the report, keeping it a measurement report: a third row, "After hygiene", in the Binaries table naming this branch's commit, and in the Results table with the same four columns; a per-subcommand count table from the shim run; and a short section "What the children are" stating whether the per-subcommand counts and the children CPU confirm the working hypothesis, that the children were mostly the per-session resize watchers' poll-mode round trips through the worktree config's tmux at two seconds per session, with the name-repair pass the remainder.
If the counts refute it, the section records the children the counts show and states that the hypothesis did not hold; no code is changed to chase the figure.
Add a deviation bullet for anything that differed from the recorded procedure, as the existing bullets do.
Every number is a single sample, as the earlier rows were, and the section says so.

Scope bound: the report's existing rows and prose stay as they are; this card appends.
