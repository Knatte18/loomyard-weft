# Fork git-mutation guard flags read-only 'git help' piped through col/grep as state-mutating

`fork-git-guard-readonly` · Note · bug

Imported from [issue #750](https://github.com/Knatte18/loomyard/issues/750).

A `burler-review` producer row (`Discussion-Burler-Review`) halted the run as `failed` with anomaly `producer-hard-failure`.
The reason lyx gave:

> review: burler: fork "<subagent jsonl>" ran a git-mutating command ("git help -m cherry-pick 2>/dev/null | col -b | grep -n -A14 \"^ *--empty\" | head -30") — a fork must never run a state-mutating git command

`git help -m cherry-pick` only displays a man page; piped through `col`, `grep` and `head` it changes no repository state.
The guard appears to match the `cherry-pick` token anywhere in the command line, not the git subcommand actually invoked (`help`).

This is abnormal: a read-only documentation lookup by the review fork hard-failed the whole run, after the Write and Bouncer rows had already completed, and discarded the review round.

What to do: make the fork's git-mutation check classify by the invoked git subcommand (`help`, `--help`, `-h` and man-page lookups are read-only) rather than by mutating verb names appearing in arguments, and add a test case for `git help <mutating-verb>`.

Provenance: task `ubuntu-2404-support`; note `loom-halt.md`.
