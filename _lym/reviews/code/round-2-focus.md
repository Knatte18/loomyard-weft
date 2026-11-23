---
round: 2
focus:
  - "Check every claim that a failed Slack toggle click is written to the trace log (the `toggle` long help in internal/slackcli/toggle.go, internal/slackcli/doc.go, _lym/landing/summary.md) against internal/logger/sink.go, which arms the durable sink only when the cwd resolves to a lym worktree: does any claim promise a log line for a click from a directory that is no worktree?"
  - "Check that _lym/landing/summary.md's blast radius names every caller of `reedengine.ShowToggleOutcome`, including `rc-on` in internal/orchcli/remotecontrol.go, and that the comment above `toggleMessageMillis` in internal/reedengine/slacktoggle.go does not name the Slack toggle as the hold's only use."
---
