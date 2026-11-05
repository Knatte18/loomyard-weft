```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge each reviewer proposal the fixer carried out (fix-proposal lens).

- **MEDIUM**: the F8 change (decision-record D4 first bullet, "a cursor keyed by channel id … A first start with no cursor for the configured channel"; D2, "reads the channel and operator id from config at each poll"; the new D4 criterion, "after a `lym slack setup` changes the channel, the relay forwards none of the new channel's history").
  - **Summary:** keying the cursor by channel replays history when setup moves back to a channel the relay used before.
  - **Evidence:** suppose the channel goes from A to B and then back to A. Channel A still has its old cursor, so it does not take the first-start path. The relay forwards every operator message posted in A while it read B, and those can be stale commands. That breaks the new criterion's "forwards none of the new channel's history". A sound change: drop any stored cursor for a channel when setup moves to a different channel, or keep only the configured channel's cursor.
- **MEDIUM**: the F3 change (D4 second bullet, "retried with the poll's backoff, without limit, so later messages wait behind it").
  - **Summary:** a creel refusal that never clears blocks every later Slack message, including the presence commands `away`, `back` and `status`, which need no creel post.
  - **Evidence:** creel refuses while any live strand's pane process is unknown (`internal/creelengine/identity.go:68-69`, `paneOfPeer`). That condition lasts as long as a strand record stays live with no pane pid. While it lasts, the operator's `status` or `back` text sits behind the stuck message and is never handled. The relay's one Slack answer gives the operator no way forward from the phone. A sound change: handle presence commands, and answers that need no creel post, even while an earlier forward waits, or bound the head-of-line wait.

The other proposals carried out (F1, F2, F4, F5, F6, F7, F9): I could not build a case that breaks them.

Further tool calls: none.
