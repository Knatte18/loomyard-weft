---
round: 1
focus:
  - "internal/reedengine/lifecycle.go and tmuxconf.go (card 3): does the sourcing happen exactly once per server start under concurrent boots, i.e. only by the boot that wins the claim, and before reed's own pins?"
  - "internal/reedengine/tmuxconf.go (card 3): is `exactSessionWindowTarget` used for every `set-option` chained after `new-session`, and does the read-back, not the guard invocation's exit status, decide whether pins applied and the session is alive?"
  - "internal/reedengine/lifecycle.go (card 3): when the operator file ends reed's session or server, is the boot refused cleanly, with no stray server or session left behind?"
  - "internal/reedengine/lifecycle.go and tmuxconf.go (card 2): is path validation run on every POSIX boot, and does the refusal match the shell refusal's place and style?"
  - "internal/reedengine/{bar,generation,lifecycle,overlay,switch,tmuxconf}.go (card 4): does every session read in Go and in the status-bar format apply the filter, with `list-sessions -f` passed off Windows only?"
  - "internal/reedengine/config.go, template_*.yaml and internal/testkit/tmuxkit/tmuxkit.go (card 1): is `tmux_conf` parsed on Windows and ignored there, and is its env variable unset in `tmuxkit.Main`?"
  - "Test economy across all four cards: do the new tests in tmuxconf_test.go, tmuxconf_integration_test.go, lifecycle_test.go and overlay_test.go duplicate each other's coverage, and does any added behavior lack a test?"
---

Seed focus from the plan overview and the commit stats.
The last item applies the rubric's test-economy rule.
