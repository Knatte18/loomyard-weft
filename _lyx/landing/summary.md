# Allow scripted edits in every agent session

Every agent session lyx spawns, the orch included, may now edit files by script (`sed -i`, `perl -i`, `awk`, or a shell heredoc or redirect) wherever that is simpler than Edit or Write.
A mechanical change across many sites, such as the planned binary rename, can now be done by script.

## What changed

- `internal/shellguard` no longer has the `in-place-edit` and `redirect-write` rows.
  Their patterns, steers and notices are gone, along with the helpers only they used (`unquotedPrefixPattern`, `projectTargetPattern`, `notPrefixPattern` and the like).
  The `python` (on its switch), `raw-go`, `refuse-raw`, `computed-rm` and `machine-sleep` rows deny exactly as before.
- The `edit-directive` stencil now permits scripted edits instead of banning them.
  It keeps the `lyx loom fmt` formatting route and the `.scratch/` rule for scratch files, and drops the `Rename`-card exception, which only existed to make room inside the ban.
  The directive is still rendered into every role's opening stencil and is still the only stencil that states the edit rule; `TestStencils_NoEditRuleRestatement` still guards that, and `editRulePhrases` no longer lists `replace_all`, which the directive no longer mentions.
- The Claude engine renders two fewer Bash hooks and two fewer notice sentences.
  The settings golden, the settings tests' deny counts and the hook-command test move with it; the hook-command test now exercises the `machine-sleep` row where it used `redirect-write`.
- The no-denied-recovery scan's self-check now asserts that `sed -i` is not a denied way forward.

## Tests

- `rows_integration_test.go` drops the removed rows' cases and adds commands that no remaining row may deny: a `sed -i` and a `perl -pi` edit, a redirect into a project file, and a heredoc appended to one, beside the existing tmux-tier reproduce command.
- `editdirective_test.go` asserts that the rendered directive offers `sed -i` and no longer bans it.

## Docs

`docs/overview.md`'s shuttle row, the `claudeengine` and `editdirective` package docs, `lyx loom fmt`'s doc and the `PATTERN-parent-directive` background file now describe the edit rule without the ban.
The `PATTERN.md` overview line is unchanged, since it names the edit directive and not the ban.

## Design decisions

- The rows are removed outright rather than put behind a config switch, because the request was to drop the ban, not to make it optional.
- The directive names the shell tools rather than "any script", so it does not steer an agent into the `python` row where that row's switch is on.
- The `python` row's steer, which suggests Edit or Write as alternatives to Python, is left as it is: it offers tools and states no ban.
- A session that started before this lands keeps the old rows and directive until it is relaunched.
