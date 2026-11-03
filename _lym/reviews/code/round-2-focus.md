---
round: 2
focus:
  - "internal/gatecli/run.go runTest and configDirs: does a test run the shared build with an env.yaml, and the hub branch of configDirs with a machine-file override, and fail if the build's own env stops winning or the machine layer is dropped?"
  - "internal/driftcli/cli.go takeRun and runCaptureChild, and internal/loomcli/driftwiring.go driftDeps: does a test fail if any of the three sites hands the run cfg.Step instead of the step from StepWithHubEnv, and is the env config load failure of StepWithHubEnv pinned?"
  - "internal/reedengine/spawn.go launchStrandLocked: does a test fail if the pane env exports no longer precede the launch command in the staged launch script, on both shell dialects?"
  - "internal/envconfig/config.go LoadConfig: what names pass validation that could shadow lym's own variables or the prelude's PATH, and does the build-server-off switch order in internal/gatecli/run.go keep lym's forced switches last?"
---

Each entry names the sites to read and the question to settle.
