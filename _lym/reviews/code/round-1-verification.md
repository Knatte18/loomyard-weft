---
round: 1
entries:
  - key: tmux-steer-omits-tier
    status: closed
    finding: F1
    reason: "d5328402d rewrites steerTmuxScriptDeny in internal/shellguard/rows.go:390 to name --tier, the package and a filter after --, without the single-test clause or quotes, and regenerates settings-golden.json to match."
---
