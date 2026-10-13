# Keep pair trace logs in a git-ignored hub archive

`run-log-archive` · Note · enhancement, batten, fabric

## Seen (2026-10-13)

A study of whether `shuttle-hook-first` had an effect could not answer: task-pair trace logs (`<pair>/.lyx/logs`) are deleted at Worktree-Teardown, so almost no shuttle or batten logs from real loom runs survive (report `.scratch/hook-first-effect.md` in the hub prime).
The volume is small: 64 to 336 KB per live pair, 1.6 MB in the hub prime over four days.

## Idea (operator, 2026-10-13)

At teardown, copy the pair's `.lyx/logs` into an archive under the hub's `.lyx/` (git-ignored by definition, never `_lyx/`), for example `.lyx/log-archive/<slug>/`, and prune it after a retention period such as 30 days.
Logs never go into the weft: git keeps them forever and every clone carries them, while they are needed for weeks.

## Open

- The retention period, and whether it is a config key.
- Whether batten's own per-slug logs and verify logs (`.lyx/verify/`) go into the same archive.
- The durable trace sink writes Info and up (Debug only when `LYX_LOG_LEVEL=debug`); `LYX_LOG_LEVEL` sets only the stderr threshold and is an environment variable, not a config key. No setting suppresses WARN in the trace files, and none is needed for the archive's size.
