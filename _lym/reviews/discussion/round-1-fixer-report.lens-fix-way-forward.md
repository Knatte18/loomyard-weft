```yaml
lens: fix-way-forward
outcome: reported
```

Task: check whether every refusal, error, next step and test the fix added can be reached and leads somewhere useful.

Findings:

1. MEDIUM — decision record D6, the new refusal bullet: "`lym slack away`, `lym slack back` and `lym slack stop` refuse … naming the operator's terminal or the Slack texts `away` and `back` as the way forward."
   For `lym slack stop` the Slack-text way forward does nothing useful.
   No Slack text stops the relay, and per D3 the Slack text `back` only sets `terminal` while the relay keeps running.
   A refused `stop` that points at the Slack texts names a step that cannot finish the stop.
   The relay must also be live for a Slack text to be read at all.
   Evidence: D4 accepts only `away`, `back` and `status` as presence commands; D3 says only `lym slack stop` ends the relay.
   The refusal for `stop` should name only the operator's terminal or an orch's pane.

2. LOW — decision record D3: "by a prime anchor change's restart step beside `EnsureDobbies` in `internal/fabriccli/anchor.go`".
   `anchor.go` brings dobbies back on two paths.
   The failure path is the `fail` closure, which calls `startDobbies(deps, ended)` at `anchor.go:386`.
   The success path is `startDobbies(deps, dobbies)` at `anchor.go:447`.
   The record does not say whether the relay is restarted on the failure path.
   If it is not, an anchor change that ended the relay and then failed leaves it down while presence is `terminal`, the case F3 exists to close.
   The D3 criterion covers only a change that succeeds.

3. LOW — decision record D5, the give-up bullet: "posts one line naming the message's id and sender and counts the message as handled".
   That line names no way to read the dropped message, though the skip line in the bullet above does (`lym creel log --to &lt;shortname&gt;:SLACK`).
   An operator on the phone gets an id with no next step.
   If the line itself fails to post, the message is dropped with nothing shown in Slack.

Further tool calls:
- One grep of `internal` and `cmd` for `PR_SET_CHILD_SUBREAPER` use: my context did not show whether any lym process is a subreaper, which would keep the double-forked relay inside a pane's tree. There is none, so the D3 double-fork bound holds within lym.
