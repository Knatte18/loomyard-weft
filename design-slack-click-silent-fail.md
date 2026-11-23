# Reed's Slack button click leaves forwarding off with no trace

`slack-click-silent-fail` · Task · bug, reed, slack · running · Drift-Verify

Imported from [issue #212](https://github.com/Knatte18/loomyard/issues/212).

On the prime's reed session, clicking the slack toggle in the status bar left `lym slack status` at forwarding [] and relay_live false, and showed nothing on the bar. No trace log of a `slack toggle` run was written. Running the same command by hand from the prime anchor, `lym slack toggle slack --socket <reed socket> --tmux /usr/bin/tmux --client /dev/pts/15`, turned forwarding on for mo:orch, started the relay and made the channel at once. The session's @lym_button_dir and @lym_orch_dir both name the prime anchor. The toggle command in @lym_slack_toggle_command sends stdout and stderr to /dev/null, so a failure before the bar redraw leaves no sign. Expected: a click flips the button, or the bar or a log shows why it did not.

## Issue comment, 2026-11-23

Likely related: bc5799ced (2026-11-17) moved the Slack toggle to the orch's session only, read through the new `@lym_orch_dir`, and its description lists a real tmux Slack click as not run by hand. The relay's last healthy time before this was 2026-11-12, so no click had been made on the new toggle until now. In a task session the bar draws the away toggle alone, and away does nothing while the relay is down.