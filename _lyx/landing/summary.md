# Allow scripted edits: drop the in-place and redirect-write guard rows

Every agent session lyx spawns, the orch included, could not edit a file by script.
The shell guard denied `sed -i`, `perl -i` and a shell redirect into a project file, and the edit directive told agents to use only Edit or Write.
The ban is no longer wanted, and the `rename-binary-lyr` rename is meant to be done mechanically by script.
After this change an agent may edit with `sed -i`, `perl -i`, `awk`, a heredoc or a redirect wherever a script is simpler.

## What changes

- `internal/shellguard`: the `in-place-edit` and `redirect-write` rows are gone from `Rows`, with their patterns and the helpers only they used.
  Their steers, deny-notice sentences and integration-test rows go with them.
- `edit-directive` stencil: it now permits scripted edits instead of banning them.
  Its `Rename`-card exception is dropped, since it only carved a hole in the ban.
  The `lyx loom fmt` route for Go files and the `.scratch/` rule for scratch files stay.
- The Claude settings document, its deny notice and `testdata/settings-golden.json` no longer carry either row's hook or sentence.
  The golden diff is large because the hook entries are renumbered, not because the rest of the settings changed.
- The no-denied-recovery scan no longer lists `sed -i` and `perl -i` as denied ways forward, because it reads the rows' `Forms`.
- Docs that described the removed rows or the ban are updated: the `claudeengine`, `editdirective` and `loomcli` package docs, `docs/overview.md`, `docs/research/codex-feasibility.md`, and the `PATTERN-parent-directive` and `PATTERN-refusal-way-forward` background files.

## Design decisions

- Both rows are removed outright, with no config switch.
  Nothing asked for a knob, and the removal applies to every session.
- The edit directive stays the one statement of the edit rule, and every opening stencil still renders it.
  `TestStencils_NoEditRuleRestatement` and `editRulePhrases` stay, reworded from the no-script rule to the edit rule.
- The directive names `sed`, `perl`, `awk` and a shell heredoc or redirect, not "any script".
  A general wording could steer an agent into the `python` row where its switch is on.
- The `python`, `raw-go`, `refuse-raw`, `computed-rm` and `machine-sleep` rows are untouched, as are Claude Code's own permission settings.
  The `python` row's steer still suggests Edit or Write; it offers alternatives and states no ban.
- The `PATTERN-parent-directive` overview line is unchanged, since it names the edit directive and not the ban.

## Check by hand

- A session started before this lands keeps the old rows and directive until it is relaunched.
- Run `lyx gate test --upto 2` on `main` before pushing; it was not run as part of this description.
