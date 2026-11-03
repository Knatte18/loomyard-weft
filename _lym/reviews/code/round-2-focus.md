---
round: 2
focus:
  - "internal/driftengine/seats.go captureReading and BaselineTable: settle whether the Drift-Baseline agent's prompt describes the capture's directory against the worktree root, with relative dir joined onto it, and renders sensibly when the step's dir is unset (no empty backticks, no mention of the removed base); check that a test asserts the prompt text for an empty and a relative dir."
  - "internal/driftengine/steprun.go missingDirRefusal and RunSideCapture: settle whether a missing dir on the side capture names the side-capture verb served by internal/driftcli/cli.go runSideCapture, not the baseline capture's lym drift capture and lym loom resume, and whether a test reaches that text."
  - "RunCapture, RunCompare, RunSideCapture in internal/driftengine and the CaptureRequest and CompareRequest constructions in internal/driftcli and internal/loomcli: settle whether each passes the worktree root as WorktreeDir and the anchor as AnchorDir, and whether a subdirectory-anchored test row would fail if one of them swapped or dropped it."
  - "internal/battencli notice and PairDir seam (wire.go, dobbynotice.go) and its integration step: settle whether the test asserts the task worktree root on a hub anchored at a subdirectory, where the worktree root and the anchor path differ."
  - "internal/configcli/hubwide.go copyStagedToHub and checkHubModule: settle whether an editor save of a drift or gate file with a missing command dir is refused and a save over a file whose dir already failed the same check is allowed, with tests on both."
---

Focus targets the one gating design finding first, then the neighbouring seams the same directory base touches.
