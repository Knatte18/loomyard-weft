# Shell guard rows read inside a sh -c or bash -c script

The command-shaped shell guard rows now read the script of a `sh -c`, `bash -c`, `dash -c`, `ksh -c` or `zsh -c` the way they read a top-level command.
A computed-path `rm` inside such a script now gets the computed-rm refusal and its way forward.
Before, it reached Claude Code's "removes files and could not be checked" dialog, which stalls an unattended run until someone answers it.

## What changed

`internal/shellguard/rows.go` splits `commandPositionPattern` into named parts and adds a script branch to it:

- `topLevelReachPattern` is the old run of shell units up to a separator.
- `commandWordPrefixPattern` is the old run of `env`, `exec`, `xargs`, `sudo` and `time` plus a path prefix.
  It now also takes the shell keywords that open a command: `do`, `then`, `else`, `if`, `elif`, `while` and `until`.
- `shellDashCPattern` matches a shell in command position followed by any flag words and then `-c`, alone or ending a cluster such as `-lc` or `-ec`.
- `singleQuotedScriptUnitPattern` and `doubleQuotedScriptUnitPattern` are the shell units inside a single-quoted or a double-quoted script.
  Each one stops at the quote that closes the script.
- `shellScriptReachPattern` puts command position at the start of the script or after a separator inside it.

Every row built on `commandPositionPattern` gains the script branch:
python, raw go, refuse-raw, shell-edit's in-place half, computed-rm, machine-sleep, headless claude, adopt take-hold and publish.
Two patterns do not build on it and get their own script branch:
the shell-edit row's redirect into a source file, and the raw go row's check for a lym environment variable name.
The raw go row's `all` word now also ends at a single quote, so `bash -c 'go test all'` is refused like `go test all`.

Package docs in `internal/shellguard/doc.go` and `internal/shuttleengine/claudeengine/doc.go` and the per-row comments are updated:
they no longer list `bash -c` as a way past the guard.
`internal/shuttleengine/claudeengine/testdata/settings-golden.json` carries the new patterns.
Its notices are unchanged.

## Design decisions

- The script is read one level deep.
  A nested `sh -c` inside a script, `eval`, `find -exec sh -c` and `xargs` with a flag before the shell still pass.
  This is the same guardrail-not-barrier standard the rows already state.
- The shell keywords were added to the command-word prefix, which goes past the record's idea.
  A loop body such as `for d in a b; do rm -rf $d; done` is a common way to write the removal from the incident, and without the keywords the script branch would not reach that `rm`.
  The keywords also apply at the top level, so `for f in a b; do sed -i s/a/b/ $f; done` is now refused too.
  Before, the shell-edit row's comment named that command as one it let through.
- A lone apostrophe inside a double-quoted script ends the scan, so a command after it passes.
  The alternative was to read every single quote as a plain character.
  That would put a quoted separator such as `'a;b'` in command position and falsely deny the command after it.

## Tests

`internal/shellguard/rows_integration_test.go` adds rows to `TestRows_PatternsDenyAndAllow`, each run through a real `grep -E` over a PreToolUse payload.
New denied rows include the incident's shape: a background `rm -rf "$d/x"` inside a `sh -c` loop over a `mktemp -d` directory.
They also cover single- and double-quoted scripts, flag clusters and flag words before `-c`, a path-prefixed shell after a separator, a command after `do`, and one row per guard row.
New allowed rows include `rm` as an argument inside a script, a literal-path `rm` inside a script, a `sh -c` script inside a quoted argument, `ssh -c`, a package-scoped `go test` in a script and `gh pr view` in a script.
The existing python row `behind_bash_dash_c` flips from allowed to denied.

## Merge danger

**Door:** two-way
**Blast radius:** shellguard
