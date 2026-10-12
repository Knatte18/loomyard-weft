# Fixer report — discussion, round 1

## Changed

All in `_lyx/discussion/decision-record.md`, with the round logged under Review rounds in `_lyx/discussion/support-log.md`.

- **F1** (BLOCKING), fixed as proposed: D1 prescribes "never `python` or `node` in an inline heredoc", matching the current directive, which `deniedRecoveryMarker` in `cmd/lyx/norecoverydeny_test.go` admits; the first acceptance criterion names `python` instead of "the interpreter heredoc".
- **F2** (MEDIUM), fixed as proposed: D6's third latch bullet says pane reads continue after the notice until an input box shows, which clears the latch; a new acceptance criterion pins a second notice for a second dialog after the first was answered.
- **F3** (MEDIUM), fixed as proposed: D6 persists the offset of the latest ask folded into the episode when the notice is sent, and attach never rebuilds from an ask at or before it; the reattach acceptance criterion covers a `PermissionRequest` followed by a `permission_prompt` `Notification`.
- **F4** (MEDIUM), fixed as proposed: D2 states the `sed` and `perl` words end at `;`, `&`, `|`, `)`, backtick or JSON-escaped newline, and that a flag word opens after whitespace with a single dash; the Bound lets through `find -exec sed -i` and the unlisted prefixes `nohup`, `command`, `timeout`, and falsely denies `perl -Mstrict -ne ...`; the in-place deny acceptance criterion admits `sed --quiet 3p f`, `sed --silent 3p f` and `sed -n 3p f` followed by a newline and `grep -i x f`.
- **F5** (MEDIUM), fixed as proposed: D7 says the closed-list entries catch only the bare `sed -i` and `perl -i`, the other D2 spellings stay outside the token scan as the raw `go` deny does, and the background file says so in the same sentence as the computed-path `rm` deny.
- **F6** (LOW), fixed by widening: D3's pattern takes `$` followed by a letter, digit, `_`, `{`, `@` or `*`, and the computed-path acceptance criterion refuses `rm $1`.
  The remaining special parameters (`$$`, `$?`, `$#`, `$!`) join the Bound's lets-through list, so the widened pattern's reach is stated too.
- **F7** (LOW), fixed as proposed: the Goal names the `sed -i` and computed-path `rm` forms; a new Auto-mode assumption says Claude Code's `rm` prompt fires only on a substitution or variable, and that a glob, `~` or a special parameter D3 lets through still stalls until D6's notice.
  The proposed assumption text listed "a positional parameter" among the forms left to Claude Code; after F6's widening D3 denies those, so the assumption names the special parameters D3 lets through instead.
- **F8** (LOW), fixed as proposed: D1's entry is the root-anchored `/.scratch/`, written through `ExcludeAnchoredDir`, with a Bound: untracked files under it escape `git status` and so a dirty-tree check, a `.scratch/` below the root is not excluded, and tracked content is never hidden.

## Disputed

None.

## Deferred

None.
