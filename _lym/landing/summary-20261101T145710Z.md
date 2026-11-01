# Refuse shell edits and steer every session to Read and Edit

Agent sessions edited files with `sed -i`, `perl -i` and heredoc redirects, often through a scratch file, at about one edit a turn.
Sessions that used the Edit tool made more than two edits a turn.
Nothing stopped a shell edit: the edit directive permitted scripted edits, and the python row's steer only suggested Edit or Write.
After this change, every session is told to read a file it changes with Read and change it with Edit, and a shell guard refuses the shell routes.

## What changes

- `internal/shellguard`: a new `shell-edit` row, installed for every run with no switch, the orch included.
  It denies `sed -i` (also `--in-place` and flag clusters such as `-Ei`), and `perl -i` (also clusters such as `-pi`), in command position after any argument words.
  It also denies a `>`, `>>` or `>|` redirect or a `tee` into a file whose extension marks a source file (Go, markdown, YAML, TOML, shell, .NET), including a redirect on a heredoc operator's line.
  Its `Forms` list `sed -i`, `perl -i` and `perl -pi`, so the no-denied-recovery scan now flags a stencil or spec that offers them.
- The edit rule now lives in one constant, `editToolsRule`: read a file with Read, whole or a span, change it with Edit with all edits of one file in one message, or write it whole with Write; grep, awk, cat and `sed -n` stay for files the session does not change.
  The `shell-edit` and `python` steers both carry it in full.
  Only the `shell-edit` notice states it in the system prompt, so the python notice is now just `lym agents never run Python.`
- The `edit-directive` stencil now states the same rule instead of permitting scripted edits.
  It names a rename across many files as one `replace_all` Edit per file, all in one message.
- `contracts/stencils`: `editRulePhrases` swaps the `` `sed -i` `` phrase for `never by shell`, because the denied-recovery scan reads `stencils.go` and now treats `sed -i` there as an offered denied command.
- The `claudeengine` settings golden file is regenerated.
  Every settings document and notice gains the new hook and sentence, so every entry changes.
- Docs updated: the package docs of `shellguard`, `editdirective` and `claudeengine`, and the shuttle entry of `docs/overview.md`.

## Design decisions

- The row has no config switch, like the computed-rm and machine-sleep rows.
  Nothing asked for a knob, and the board entry wants the refusal to cover every session.
- "Tracked source file" is approximated by extension, because a grep over the payload cannot ask git.
  `.json`, `.txt` and `.log` targets pass, so output captured into a scratch file, such as `lym board list > .scratch/board.json`, is not refused.
  A redirect into a source-extension file under `.scratch/` is refused, and Write is the way forward there.
- `tee` is refused alongside redirects, because it is the next route a session takes once a heredoc redirect is refused.
- This brings back a refusal of scripted edits that an earlier change removed so a rename could be done by script.
  The board entry (2026-11-01) is newer and asks for the ban explicitly, so no parent decision was sought.
  A large rename now goes through Edit with `replace_all`.

## Limits

The row is a guardrail, not a barrier.
An edit behind `bash -c`, a script, `awk`, `cp` or `mv` passes.
So does a redirect into a quoted target or a file of another extension, and anything after a heredoc operator's line.
A session started before this lands keeps its old settings and directive until it is relaunched.
