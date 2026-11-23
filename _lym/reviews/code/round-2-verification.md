---
round: 2
entries:
  - key: tmux-row-hangs-on-autonomous-switch
    status: closed
    finding: N1
    reason: "755895f7b replaces SwitchAutonomous with SwitchDenyRawTmux fed by the new Spec.AllowRawTmux in claudeengine settings.go and codexengine guards.go, never by Interactive, and the sampler spec sets it."
  - key: gate-test-lacks-run-agent-tmux-tier-refusal
    status: closed
    finding: N2
    reason: "44137e1bf and 47888d3f5 add the run-agent tmux-tier refusal in gatecli run.go flagShapeRefusal, before any slot, naming the single-test way forward, with a cli_test.go test and a doc.go entry."
---
