# Fork git-mutation guard flags read-only 'git help' piped through col/grep as state-mutating

`fork-git-guard-readonly` · Task · bug, burler · running · Darn-Burler-Review

Imported from [issue #750](https://github.com/Knatte18/loomyard/issues/750).

A `burler-review` producer row (`Discussion-Burler-Review`) halted the run as `failed` with anomaly `producer-hard-failure`.
The reason lyx gave:

> review: burler: fork "<subagent jsonl>" ran a git-mutating command ("git help -m cherry-pick 2>/dev/null | col -b | grep -n -A14 \"^ *--empty\" | head -30") — a fork must never run a state-mutating git command

`git help -m cherry-pick` only displays a man page; piped through `col`, `grep` and `head` it changes no repository state.
The guard appears to match the `cherry-pick` token anywhere in the command line, not the git subcommand actually invoked (`help`).

This is abnormal: a read-only documentation lookup by the review fork hard-failed the whole run, after the Write and Bouncer rows had already completed, and discarded the review round.

What to do: make the fork's git-mutation check classify by the invoked git subcommand (`help`, `--help`, `-h` and man-page lookups are read-only) rather than by mutating verb names appearing in arguments, and add a test case for `git help <mutating-verb>`.

Provenance: task `ubuntu-2404-support`; note `loom-halt.md`.


## From issue #751

Imported from [issue #751](https://github.com/Knatte18/loomyard/issues/751).

Seen in ubuntu-2404-support, 2026-10-19 17:08: Discussion-Burler-Review round 1 failed hard (producer-hard-failure) because a review fork ran `git help -m cherry-pick 2>/dev/null | col -b | grep -n -A14 "^ *--empty"` to read the cherry-pick man page, and burlerengine's forkViolation rejected it as a state-mutating git command.

Cause: `mutatingGitPattern` in internal/burlerengine/cluster.go matches a mutating verb anywhere among git's later words (`git(?:\s+[^\s;&|]+)*?\s+(?:add|commit|...|cherry-pick|...)`), not only in the subcommand position, so `git help cherry-pick`, `git log --grep merge` or `git show tag`-style reads are refused too.

Workaround: none by hand; the run's Go driver and arbiter take the failed row.

Expected: only the subcommand word (the first non-option word after git and its global options such as -C <dir>) decides whether a git command mutates; read-only subcommands such as help, log, show, diff, status and grep pass whatever their arguments name.