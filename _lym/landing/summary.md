# shellguard: read the script of a shell -c call like a top-level command

A `sh -c`, `bash -c`, `dash -c`, `ksh -c` or `zsh -c` script that removed files by a computed path reached Claude Code's own dialog instead of the guard's refusal.
The dialog stalled an unwatched run until someone answered it by hand.
Seen in `hardener-campaign`: a fixer's loop inside a `bash -c` script made a temp dir and removed a subdirectory of it by a variable path.

`internal/shellguard` now reads the script of a shell `-c` call in command position, one level deep, and applies the command-shaped rows to it.
The agent gets the row's refusal with its way forward, and no dialog opens.
The rows that read inside the script are `python`, `shell-edit`, `computed-rm`, `machine-sleep`, `headless-claude`, `adopt-take-hold`, `publish`, `raw-go` and `refuse-raw`.

## What changes

- `commandPositionPattern` in `internal/shellguard/rows.go` now covers three places a command can stand: the top level, a single-quoted script and a double-quoted script.
  Each place has its own reading of quotes, so a row's argument scan stops at the script's closing quote and never runs on into a later command.
  `shellScope` carries that per-place reading, and `commandRowPattern` renders one alternative per place.
- The `shell-edit` redirect pattern and the `raw-go` variable-name patterns read inside the script too.
- The shell keywords `do`, `then`, `else`, `if`, `elif`, `while` and `until` now open a command position at the top level as well, beside `env`, `exec`, `xargs`, `sudo` and `time`.
  A top-level `for f in a b; do rm $f; done` is therefore refused, and so is a `sed -i` after `do` or `then`.
- The package doc, the row comments and the `claudeengine` doc state the new reach.
  The pass lists no longer name `bash -c`; they name a script file, a nested `bash -c` and `eval`.
- `rows_integration_test.go` pins the new behaviour with rows for each guard: a denial inside a script, and a pass for the same word as an argument, inside an inner quoted string, in a later command after the script, or inside a quoted argument that only mentions `bash -c`.
- `testdata/settings-golden.json` carries the regenerated patterns.

## Design decisions

- Keyword positions widen the top-level rows too, not only the script scan.
  A removal by a computed path inside a top-level loop stalls a run on the same dialog as one inside `bash -c`, so one rule serves both.
  A keyword only marks a command position, so it adds no refusal for a command the guard already allows.
- The scan is one level deep.
  A script behind a nested `bash -c`, `eval`, `find -exec` or a flag of `xargs` still hides its commands, and a lone single quote inside a double-quoted script hides the commands after it.
- Inside a double-quoted script, a lym variable expanded in an inner double-quoted string is not read by the `raw-go` row.
  The doc comments state this exception.
- The guard stays a guardrail, not a barrier: it greps static text.

## To check by hand

- `internal/shuttleengine/claudeengine/doc.go`: confirm the guard section no longer describes `bash -c` as passing wherever the rows now read inside it.
  The section's pass lists were reworded in this change, and one was flagged as stale.
- `internal/shellguard/rows.go`, `commandWordPrefixPattern`: confirm the keyword widening at the top level is the intended scope.
  It changes top-level refusals for `do`, `then`, `else`, `if`, `elif`, `while` and `until` as well as scripts.
- `internal/shellguard/rows.go`, the scope's argument scans (`argument`, `goArguments`, `mutationScan`): confirm a row's scan cannot run past the script's closing quote into a later command.
  The rows pin the cases listed above; a spelling outside them is unproven.

## Merge danger

**Door:** two-way
**Blast radius:** shellguard
