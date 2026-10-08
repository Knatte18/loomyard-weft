---
round: 2
focus:
  - "Run `go test -tags tmux ./internal/reedcli/ -run TestSmokeDotFill` on HEAD and settle whether the CrossClientControl scenario, which attaches a short toucher client and waits for the first strand's marker, passes under the two status rows and the pane-border titles the branch adds, and whether its sizing comment matches the pane size it uses."
  - "Run `go test -tags tmux ./internal/loomcli/ -run TestIntegrationDriverBootstrap` on HEAD and settle whether the stub driver script in `integrationWriteStubDriverScript` consumes the typed lines in the order `internal/shuttleengine/run.go` `Runner.start` types them (a `/color` line when the strand has a color, then the skill-load message, then the prompt pointer), and whether its doc comment describes that order."
  - "Settle whether any comment in `internal/reedengine` (`lifecycle.go`, `doc.go`) or any test header in `internal/burlercli` still describes the deleted status-line template, `ValidateStatusLine` or `Geometry.RepoName`, or states where boot, launch and the attach pre-flight set the `@lyx_strands`, `@strand` and `@strand_color` pane options differently from what the code does."
---

The focus entries ask for fresh verification of the fixed text, since a fix is new text no reviewer has seen.
