# Shell guard: match commands, not quoted text; refuse redirect writes

The shell guard in `internal/shellguard` now matches each row on the command that runs, and refuses shell redirect writes to project files like the other scripted-edit cases.

Before, a row matched its pattern anywhere in the command text.
A `grep` for a refused tier's flag string, a heredoc that drafted test code, or a commit message in double quotes naming a module-wide `go test` was refused as if it ran the test.
A `cat >> docs/x.md <<EOF` append went through, although the edit directive sends project-file writes through Edit or Write.

Closes #770
Closes #772

## What changes

- Every command-shaped row now anchors at the payload's `"command"` key and finds command position by a quote-aware scan, left to right.
  The scan reads unquoted characters, JSON escape pairs, single-quoted strings, double-quoted strings, and at most one heredoc operator, after which it stops at the end of that line.
  A row's command word matches only directly after a separator the scan reached that way.
- The raw go row refuses the slot, strand and prebuilt-binary variable names when they are named outside quotes or expanded inside double quotes.
  Naming one in a single-quoted or double-quoted grep pattern or in a heredoc body is no longer refused.
- A new row, `redirect-write`, is installed for every run with no config key.
  It refuses a `>`, `>>`, `>|` or `&>` redirect whose target is not under `.scratch/` by its relative path, `/tmp/` or `/dev/`.
  A file-descriptor duplication such as `2>&1` and a target starting with a variable pass.
- The claude engine shell-quotes each pattern into its hook command instead of wrapping it in bare single quotes, so a pattern may hold a single quote.
  `internal/shuttleengine/claudeengine/testdata/settings-golden.json` is regenerated for the new rendering; most of its diff is that.
- The `internal/shellguard` and `internal/shuttleengine/claudeengine` module docs state the quote-aware rows, the redirect-write row and the pattern contract.

## Design decisions

- One shared command-position prefix serves the whole table, so every row has one notion of command position.
  The guard stays `grep -E` patterns over the PreToolUse payload; no shell parser or hook binary was added.
- `redirect-write` cannot tell a tracked file from an untracked one, so every path outside the scratch places counts as a project file.
  `/tmp/` is allowed although the directive asks for `.scratch/`, since it holds no project file.
- The new row's steer names no denied command as a way forward, and steer and notice text hold no quote or backslash.

## Known bounds

The rows stay guardrails, not a sandbox.

- The scan sees nothing inside quotes, after a heredoc operator's line, or behind an escaped double quote inside a double-quoted string it misreads, so a command there passes.
- `redirect-write` lets through `tee`, `cp`, `dd`, a write behind `bash -c` or a script, a redirect after a heredoc operator's line, a target behind a variable, and an absolute path into `.scratch/`.
- It falsely denies an untracked file outside the scratch places, and a `>` comparison inside `[[ ]]` or `$(( ))`.

## Check by hand

- Run `lyx gate test --tags integration ./...`; this description's author did not run it.
- Watch a hook run on a large heredoc payload: a `grep` without a DFA engine could run the quote-aware pattern slowly there.
