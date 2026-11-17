---
round: 2
focus:
  - "internal/orchcli/remotecontrol.go and internal/orchcli/start.go: where the orch's engine and prompt engine come from; settle whether any caller names an engine literal rather than reading config, per PATTERN-code-conventions."
  - "internal/dobbyengine/tidy.go and internal/beamcli/dobbydeps.go: when the shuttle runner cannot be built, settle whether the build error reaches the caller of spawnTidy, or a missing engine/edit-directive stencil error masks it."
  - "internal/testkit/stencilkit/stencilkit.go and internal/editdirective: settle whether the edit-directive block name is declared once, and whether an import cycle is avoided by the test package layout."
  - "Producer tests that assert PromptEngine (treadleengine, samplerengine, landingshed, frictionengine, mergeresolve, dobbyengine, shedarbiter, sheddriver, loomcli, weaverengine, websterengine, gatefixer, shedadapters, seatengine): settle whether each asserts the rendered prompt holds the edit-directive copy of the engine its launch resolves to, under a Codex namer."
  - "internal/loomcli/driverprompt.go: settle whether the loom driver's prompt render function is exported and takes the engine name, as every producer outside seatengine must, and whether the change description matches."
---

Focus targets the places the round's gating findings touched, so a fix there is read as new text.
