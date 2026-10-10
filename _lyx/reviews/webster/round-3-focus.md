---
round: 3
focus:
  - "internal/loomcli/integration_driverbootstrap_test.go: the stub script `integrationWriteStubDriverScript` and its comments. Settle whether, with `driverSkills` empty in internal/loomcli/driverspec.go, the stub takes the prompt.md path from its own argv (the claudeengine launch-line pointer) and no longer reads typed skill-load or pointer lines, and whether the test passes under `lyx gate test --tags tmux ./internal/loomcli`."
  - "Comments in internal/loomcli/integration_driverbootstrap_test.go describing the launch flow (the driver spec's skill list, Start's paced typing, `stubSettleDelay`): settle whether each matches the current flow, where claudeengine.Prepare puts the pointer on the launch line when the spec names no skills."
---

Other tmux-tagged tests that depend on the driver spec's skill list are worth a check for the same stale-flow cause.
