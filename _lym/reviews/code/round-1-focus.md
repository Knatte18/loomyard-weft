---
round: 1
focus:
  - "internal/reedengine/bar.go, buttonrow.go, panebin.go: does the Slack button draw only in a prime's or a beam's session and never in a task session, and does the bar's doc comment match what is drawn?"
  - "internal/reedengine/remotecontrol.go and bar.go: does the RC button resolve the agent of the window's active pane, and does a click turn remote control on for that agent alone through creel's remote-control command, never the parent orch?"
  - "internal/orchcli/remotecontrol.go, start.go, cli.go: after the rewrite, does RC stay off for every agent until its button is clicked, with orch.yaml's remote_control for orch sessions unchanged, and are failure paths (unresolvable agent, queue failure) handled and reported?"
  - "internal/orchengine/remotecontrol.go and state.go removals, plus orchroute/paths.go: is no caller or reference to the removed state pin and helpers left, in code, docs and doc.go files?"
  - "internal/reedengine/bindings.go and the *_test.go set-option counts: do the changed expectations follow from the behavior change, and does any new test duplicate an existing one's coverage or leave added behavior uncovered?"
---

Seed focus from the decision record's Decided list: Slack stays per orch, RC moves to the active pane's agent, RC defaults off, and the bar's doc comment matches the drawing.
