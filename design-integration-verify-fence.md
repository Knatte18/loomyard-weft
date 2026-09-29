On the first real lyx loom run (task `burler-exclude-warn`, PR #280), Webster's integration fork reported that the verify command in its rendered prompt `integration.md` was mangled by nested code fences, so it ran the plan's own `## verify:` command from `00-overview.md` instead.

The fork recovered by itself, but the integration stage's prompt carries the one command that decides whether the whole plan passes: a fork that trusts the rendered text runs something other than the plan's verify, and one that cannot recover it fails the stage.

Expected: the verify command renders into `integration.md` verbatim and unambiguous, whatever fences the plan's command or its surrounding prompt text use (e.g. a fence longer than any backtick run inside the command, or the command outside a fenced block).

Source: https://github.com/Knatte18/loomyard/issues/285 (close it in the landing commit).