# Allow scripted edits in every agent session

Every agent session lyx spawns, the orch included, may now edit files by script (`sed -i`, `perl -i`, `awk`, or a shell heredoc or redirect) wherever that is simpler than Edit or Write.
A mechanical change across many sites, such as the `rename-binary-lyr` rename, can be done by script instead of one Edit per site.

## What changes

- `internal/shellguard` no longer has the `in-place-edit` and `redirect-write` rows.
  Their patterns, steers and notices go with them, as do the helpers only they used.
  The `python` (on its switch), `raw-go`, `refuse-raw`, `computed-rm` and `machine-sleep` rows deny as before.
- The `edit-directive` stencil permits scripted edits instead of banning them.
  It keeps the `lyx loom fmt` formatting route and the `.scratch/` rule for scratch files.
  It drops the `Rename`-card exception, which only existed to carve a hole in the ban.
- The Claude engine renders two fewer Bash hooks and two fewer deny-notice sentences, so `settings-golden.json` and the settings tests' counts move with the rows.
- The no-denied-recovery scan no longer lists `sed -i` as a denied way forward, and its self-check asserts that.
- Docs that described the rows or the ban now describe the edit rule: `docs/overview.md`, the `claudeengine`, `editdirective` and `loomcli` package docs, `docs/research/codex-feasibility.md`, and the `PATTERN-parent-directive` and `PATTERN-refusal-way-forward` background files.
  The `PATTERN.md` overview line is unchanged, since it names the edit directive and not the ban.

## Closes

No issue.

## Design decisions

- The rows are removed outright, with no config switch: the request was to drop the ban, not to make it optional.
  The removal lets every in-place `sed`/`perl` edit and every shell redirect into a project file through, in every session.
- The edit directive stays the one statement of the edit rule, now permissive.
  `TestStencils_NoEditRuleRestatement` and `editRulePhrases` stay, reworded from the no-script rule to the edit rule.
- The directive names `sed`, `perl`, `awk` and a shell heredoc rather than any script, so it does not steer an agent into the `python` row where that row's switch is on.
- The rename-tool route a Webster implementer follows for a symbol `Rename` card is untouched.

## To check by hand

- A session started before this lands keeps the old rows and directive until it is relaunched.
- Neither a `sed -i` nor a redirect into a project file was run in a live agent session; the coverage is the row table's tests and the pinned settings golden.
