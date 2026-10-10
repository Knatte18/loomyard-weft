---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: consistency
    location: internal/loomcli/smoke_driverstrand_test.go:61
    summary: The smoke stub script and its doc comment still describe the retired skill-load flow, which the driver spec no longer has
---

### [LOW] Smoke stub still models the skill-load flow

**Location:** `internal/loomcli/smoke_driverstrand_test.go:61-95` (`writeStubDriverScript` and its doc comment).

**Issue:** `driverSkills` is empty (`internal/loomcli/driverspec.go:18`), so `claudeengine.Prepare` puts the prompt pointer on the launch line and shuttle types only the `/color` line.
The doc comment still says "The driver spec names skills, and shuttle types one skill-load message before the prompt pointer", "answers skill loads", and that the script "starts the sleep at the next line, the pointer".
The script body follows that flow: after `/color` it reads one more line, then appends a Stop event to `events.jsonl` and reads a third line before `sleep 3600`.
No second or third line is ever typed, so the script blocks in `read` and never reaches the `sleep`.
The test still passes only because a stub blocked in `read` keeps its pane alive, which is the property the test needs; the script no longer does what its comment says.
This is the same stale-flow cause round 3's focus names for `integration_driverbootstrap_test.go`, in the other tmux-tagged test that depends on the driver spec's skill list.

**Fix:** Drop the `--settings` argument scan, the `case $line in /color*)` second read, the Stop-event append and the third read; after reading the one `/color` line and printing the fixture, go straight to `sleep 3600`.
Rewrite the doc comment to say the pointer rides the launch line and the only typed line is `/color`.

## Focus checks

- Focus 1, `integrationWriteStubDriverScript` in `integration_driverbootstrap_test.go`: with `driverSkills` empty, the stub takes the `prompt.md` path from its own argv (`case $1 in *prompt.md*)`), reads only the one typed `/color` line, and no longer waits for a skill-load or a typed pointer.
  `lyx gate test --tags tmux ./internal/loomcli` passes at HEAD (`ok ... internal/loomcli 130.421s`, exit 0), so `TestIntegrationDriverBootstrap_ReturnsWithoutWaitingOnTheDriver` no longer fails and I raise no finding for it.
  Shuttle does play the color sequence after startup readiness and, with no skills, before nothing else (`internal/shuttleengine/doc.go:148-151`), which matches the one-line stub.
- Focus 2, comments in that file: the stub's doc comment, the `stubSettleDelay` comment ("startup probing and its typing and settling of the color command") and the surrounding description of the launch flow all match the flow `claudeengine.Prepare` produces when the spec names no skills (`claudeengine.go:172-175`).
  No stale comment remains in that file.
- The other tmux-tagged test depending on the driver spec's skill list is `smoke_driverstrand_test.go`, which is F1.

The rest of the range (cards 1 to 20 and the fix commits) was not re-derived beyond the focus: the rubric's Do-not-flag list covers the plan's own gates, and this round's focus is the Publish failure and its stale-flow cause.
