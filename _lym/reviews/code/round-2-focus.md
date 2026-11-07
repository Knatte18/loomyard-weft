---
round: 2
focus:
  - "internal/reedengine/lifecycle.go logBootWindowLocked: does a test in the package assert the Info line for the boot window carries window id, pane id and name, and that a failed or single-token display-message answer leaves the boot successful (booted=true, no error)?"
  - "internal/reedengine/doc.go window-order section: does the text about how reed opens windows match the code, given the boot's first window comes from new-session in serverSpawnArgv without -n and is renamed by the shell-window step?"
  - "internal/reedengine/spawn.go openOwnWindowLocked and ownwindow.go moveShellWindowStrandsLocked: is the name field on their Info lines covered by an existing log-capture test, or is it correctly left log-only because the package has no capture harness?"
---

Round 2 re-checks the sites above in the fixed tree; the rest of the diff is settled.
