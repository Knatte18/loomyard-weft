# Agents change files with Edit, never shell edits; a guard refuses them

Agents in lym sessions edited mostly by shell: `sed -i`, often in two steps through a scratch file, at about one edit a turn, where the Edit tool made more than two edits a turn.
Nothing refused a shell edit, the steer sentence named `sed -n` and `awk` for reading, and Claude Code's bypass-mode reminder, which lym's launches carry, tells agents they may read with `cat`, `head` or `sed -n` and change files with `sed`, heredocs or short scripts.
This change makes the Edit tool the way a session changes files.

## What changes

- A new `shell-edit` row in `internal/shellguard/rows.go` refuses a Bash call that runs `sed -i` (including `--in-place` and clusters such as `-Ei`), `perl -i` (including `-pi`), `tee` into a source file, or a `>`, `>>` or `>|` redirect into a source file.
  The row is installed for every run, the orch included, with no config key.
  A source file is one ending in `.go`, `.mod`, `.sum`, `.md`, `.yaml`, `.yml`, `.toml`, `.sh`, `.ps1`, `.cs`, `.csproj`, `.sln`, `.props` or `.targets` (`sourceFilePattern`); a `.json`, `.txt` or `.log` target passes.
- The steer is now shared by the python and shell-edit rows through `editToolsRule`: read a file you change with Read, whole or a span through offset and limit, edit it with Edit with all edits of one file in one message, write a whole file with Write, and search with grep.
  It no longer names `sed -n`, `awk` or `cat`.
  The system prompt states the edit rule once, in the shell-edit row's notice; the python row's notice now only announces the python deny.
- The edit directive (`contracts/stencils/edit/edit-directive.md`) replaces the permission to edit by script with the rule above, states that in lym sessions it overrides any harness text permitting shell reads or edits, names `replace_all` for a rename across many files, and says a scratch file with a source extension is written with Write.
  `internal/editdirective/doc.go`, `internal/shellguard/doc.go`, the claude engine's `doc.go` and `docs/overview.md` follow.
- The claude engine's settings golden (`testdata/settings-golden.json`) carries the new hook row; its tests and the shellguard unit and integration tests cover the new row and the changed steer.

## Design decisions

- The guard is a guardrail, not a barrier, as the code comment on the row says: an edit behind `bash -c`, a script, `awk`, `cp` or `mv`, after `do` or `then`, behind `find -exec` or `xargs` with a flag, a redirect into a quoted target or into an extension outside the list, and anything after a heredoc operator's line pass.
  The aim is to remove the habit, not to sandbox the shell.
- Captured command output into `.json`, `.txt` or `.log` stays allowed so scratch output is not blocked; a scratch file with a listed extension must be written with Write.
- The harness override sits in the directive and in the steer wording, since the bypass-mode reminder is not under lym's control.

## Check by hand

- The directive overrides the harness's bypass-mode text only by stating so: confirm in a live session, such as a forked run, that an agent given the directive reads with Read and edits with Edit rather than following the harness text.
- The steer must name Read and grep and no longer `sed -n`, `awk` or `cat`: check the denied-call text a session shows after a `sed -i` and after a Python call.
- The guard falsely denies a matching spelling after a separator inside a quote that `commandPositionPattern` misreads; skim `internal/shellguard/rows.go` for whether that false-deny case is acceptable.
- Run the shellguard and claude engine integration tests (`lym gate test ./internal/shellguard ./internal/shuttleengine/claudeengine` with the integration tier), which exercise the hook against real payloads.
