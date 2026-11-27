---
round: 2
focus:
  - "Config key naming across the diff's messages, comments and docs (internal/dobbyengine config.go, keynames.go, doc.go, template.yaml, tasksetting.go, runwatch.go, gatehold.go; internal/beamsetting/beamsetting.go; internal/battencli/cleanup.go help): is every config key written in full and rendered from a named constant, per PATTERN-config-key-names and PATTERN-code-conventions?"
  - "Config files in new and changed tests, internal/beamcli/dobbydeps_integration_test.go first: does each test write another module's config only the way PATTERN-test-stand-ins allows?"
  - "internal/dobbyengine/runwatch.go and internal/battencli/dobbynotice.go: when the child status changes during a tick, or no child status exists yet, does one stop post exactly one line, with the reason's paths relative to the pair?"
  - "internal/dobbyengine/gatehold.go, config.go and internal/beamcli's gate-hold wiring: what does a notice name on each platform, which holders count as inside the task worktree, and which gate_hold_factor values are accepted?"
  - "Tests of the run watch, the done-run clear and the gate-hold notice (internal/dobbyengine runwatch_test.go, donerun_test.go, gatehold_test.go; TestDobbyGateHolders): for each behavior the diff adds, would a test fail if it were removed, with expected values independent of the code under test?"
---
