# Fixer report — discussion round 2

## Changes

- F1 (MEDIUM), fixed as proposed: D5's third bullet now names every channel that types into the session's pane (the operator, `lyx shuttle send`, `lyx creel inject`, the `lyx creel post` reminder line) and the bound each carries: send's pause refusal, creel's inject rule for EXTERNAL callers, programs, orch strands and other strands, any strand's post starting a turn, and creel's operator-user identity boundary at the 0700 lyxtemp root.
  It also adds that a covering pause holds creel's deliveries (`internal/creelengine/doc.go` line 118), so the pause bound is stated for every automated channel.
- F2 (LOW), fixed as proposed: D1 says the empty mode starts the session in the settings' default mode, prompting when none sets one; D3 says `prompt` cannot override a settings default mode; D5's first bullet says lyx turns bypass on only through the flag while a settings `defaultMode` can still set it outside lyx; D5's fourth bullet says bypass replaces the settings' default mode with skipping every dialog.
  The fourth bullet's "in prompt mode" became "without the flag", since an opened session today runs in the settings' default mode rather than prompting.
- F3 (LOW), fixed as proposed: D5's second bullet now says the session loads Claude Code's own settings layers, the worktree's included, and that lyx installs no rule or hook of its own.

## Disputed

None.

## Deferred

None.
