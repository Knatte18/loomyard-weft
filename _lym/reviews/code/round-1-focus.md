---
round: 1
focus:
  - "internal/shuttlecli/hookpython.go: does the hook read the hub's current deny_python on every call, through the same layered config resolution `lym config --resolved shuttle` uses, so a flag flipped in the hub's layers takes effect without a restart?"
  - "internal/shuttlecli/hookpython.go failure paths: what happens when the hub config is unreadable, no hub is found, or the value is missing; does the hook fail open or closed, and is that consistent with the other guard rows?"
  - "internal/shuttleengine/claudeengine/settings.go and codexengine/guards.go: is the python row now always installed with the value no longer baked in at launch, while the system prompt notice still follows the value at launch as the decision asks?"
  - "The original report: do the launch paths of `lym orch` and other roles hand the shuttle config loader the hub's layers, so `deny_python: false` at the default layer drops the notice; is the asked-for orch-launch test present and does it exercise that?"
  - "claudeengine and codexengine parity: do both engines apply the same live-read behavior, and does the codexengine guard command resolve the hub the same way as the claude hook?"
  - "Scope: do the changes to internal/shuttleengine/config.go, engine.go, template.yaml, docs/overview.md and the doc.go files go beyond what the decision record licenses, or leave a stale statement that deny_python is baked in at launch?"
  - "Test economy: check TestBuildSettings_DenyToggleMatrix, listed as contained in the redundancy report, and TestInstalledGuards_CoverEveryRow, against their covering tests' assertions before filing."
---

The record is a decision record alone: the board entry under `## Goal` plus one amendment, which makes the hook read deny_python live.
