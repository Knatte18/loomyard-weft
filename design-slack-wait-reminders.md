# Dobby reminds the operator on Slack of a run waiting at PR-Review or Sampler-Session

`slack-wait-reminders` · Task · feature, reed · running · PR-Review

Filed from the Sketch `reed-slack-relay`.

Takes: slack-relay's presence reader, remind_after_min loader and SLACK program mailbox; dobby's probe-tick child readings and its tolerant state record; pauserecord's resumed entries
Delivers: a once-a-minute check in every worktree's dobby that keeps each PR-Review or Sampler-Session wait's start in dobby's state record and posts one reminder line per interval to <shortname>:SLACK while presence is slack
Exports: pauserecord's latest-resumed-entry lookup for a slug

The part changes these packages:

- internal/dobbyengine (about 250 changed lines)
- internal/beamcli (about 40 changed lines)
- internal/pauserecord (about 40 changed lines)
- internal/slackengine (about 20 changed lines)
