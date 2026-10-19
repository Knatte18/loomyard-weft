# Fixer-report, round 1

## Changes

All in `internal/burlerengine/cluster.go` and `cluster_test.go`; commits `4cd93dfc4` (F1–F5, one commit since they share one rewrite) and a follow-up for the self-check findings.

- F1: fixed as proposed. `-h`/`--help` is a help request only among the global options or directly after the subcommand (`gitSubcommand` now returns the subcommand, the words after it and a help flag). Rows for `git commit -m -h`, `git tag -m --help v1` and `git checkout -- -h`.
- F2: fixed as proposed. `splitShellSegments` tokenizes with single and double quotes. Rows for `-c "user.name=A B"`, `-C "/path with space"` and the single-quoted form.
- F3: fixed with an own change. A backslash-newline pair is removed, not turned into a space, which matches bash (`git com\` newline `mit` is `git commit`); the segment split runs inside the same tokenizer. Rows for a continuation before the subcommand and inside a word.
- F4: fixed as proposed. `--attr-source` added to `gitValuedGlobalOptions`, with a row.
- F5: fixed as proposed. Rows for `--git-dir=x` and `--work-tree x`.

## Self-check

- fix-code-correctness, MEDIUM: an escaped quote (`git -c "a\"b" commit`, `echo it\'s; git commit`) swallowed later text. Fixed as proposed: a backslash escapes the next character outside single quotes. Two rows added.
- fix-proposal, MEDIUM: an apostrophe in a `#` comment or an escaped quote opened a quote that swallowed later segments. Fixed as proposed for escapes (above) and with an own change for comments: a `#` at the start of a word skips to the end of its line. Rows added.
- fix-proposal, LOW: a backslash-newline inside a word must join it. Fixed as proposed (see F3).
- fix-proposal, LOW: help escape counterexample attempt for F1. No breakage found; no change.
- fix-consistency and fix-affected-set: not spawned. Their findings are none; the changed code has one caller and one test matrix.

## Gate repair

The gate refused the round for fixed-column-wrapped comment lines in `cluster.go`, on `splitShellSegments` and `gitSubcommand`.
Both doc comments are now one sentence per line, in a further commit; no behavior changed.

## Disputed

None.

## Deferred

None.
