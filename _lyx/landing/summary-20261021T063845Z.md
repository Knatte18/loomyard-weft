# Shell guard reads commands outside quotes and refuses redirect writes

The shell guard in `internal/shellguard` matched its command words anywhere in the PreToolUse payload.
So a quoted `grep` pattern or a heredoc body naming a tmux-tier `go test` or one of lyx's environment variables was refused as if it ran something (#770).
It also let shell redirect writes to project files through, though the edit directive sends those through Edit or Write (#772).

## What changes

- **Quote-aware command position.**
  `commandPositionPattern` now anchors at the payload's `"command"` value and scans it as the shell reads it, one unit at a time: unquoted text, JSON escape pairs, single-quoted strings, double-quoted strings, and at most one heredoc operator with the rest of its line.
  A row's command word matches only after a separator reached that way, so text inside quotes, heredoc bodies and other payload fields such as `description` no longer trips the python, raw go, in-place edit, computed-rm or machine-sleep rows.
  Separators are `; & |`, a backtick, `(`, `<(`, `>(` and a JSON-escaped newline.
  After a heredoc operator, only a separator on that operator's line counts.
- **Raw go variables.**
  The slot, strand and prebuilt-binary variable names are refused when named outside quotes or expanded inside a double-quoted string, and no longer inside a quoted grep pattern, a single-quoted string or a heredoc body.
  The steer now says a command "setting or reading" one is refused.
- **New `redirect-write` row.**
  Installed for every run, with no config key.
  It refuses a `>`, `>>`, `>|` or `&>` redirect, reached by the same scan, whose target is not under `.scratch/` by its relative path, `/tmp/` or `/dev/`.
  A redirect on a heredoc operator's line counts, so both `cat >> docs/x.md <<EOF` and `cat <<EOF > docs/x.md` are refused.
  File-descriptor duplication such as `2>&1` and a target starting with a variable pass.
  `notPrefixPattern` builds the "departs from this prefix" alternations, since grep -E has no negation.
- **Engine quoting.**
  The scan needs to name the single quote, so the claude engine's `renderShellGuard` now wraps each pattern with `shQuote`, not bare single quotes.
  The package doc's payload contract says a pattern may hold a single quote.
  `testdata/settings-golden.json` is regenerated for the new patterns, the new row and the steer wording.

## Design decisions

- The guard stays grep -E over static text.
  A Go-side hook with a shell parser would be exact, but every Bash call would then have to spawn `lyx`.
  The scan is a guardrail: it misreads a JSON-escaped double quote inside a double-quoted string and a quote inside a comment, and it sees nothing after a heredoc operator's line.
  Each pattern's comment lists what it lets through and what it falsely denies.
- grep cannot tell a tracked file from an untracked one, so `redirect-write` treats every path outside the scratch places as a project file.
  An absolute path into `.scratch/` is refused too, because expressing "contains `/.scratch/`" as a negation in grep -E is long and fragile.
  The steer names the relative `.scratch/` path as the way forward.
- `/tmp/` is allowed because it holds no project file, though the edit directive asks for `.scratch/`.
- Writes through `tee`, `cp`, `dd`, `bash -c` or a script stay out of scope.
  The board entry names heredoc and redirect writes only.

## Tests

- `internal/shellguard/rows_integration_test.go` adds rows for the false denies that are now let through: quoted grep patterns, a double-quoted commit message, heredoc bodies and single-quoted variable names.
  It adds rows for the forms that stay refused: a subshell, separators after closed quotes, a separator on a heredoc operator's line, and double-quoted variable expansions.
  It adds the `redirect-write` deny and allow cases.
  It now shell-quotes the pattern it greps.
- `internal/shuttleengine/claudeengine/hookcmd_integration_test.go` runs the rendered commands under `sh` for a pattern holding a single quote and for a row with several patterns.
- `settings_test.go` counts the new always-installed row.
