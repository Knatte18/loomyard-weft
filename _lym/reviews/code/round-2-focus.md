---
round: 2
focus:
  - "internal/beamcli/dobbydeps.go (tellRefusal) and internal/beamcli/keynames.go: does every message the diff adds or changes about a refused autonomy value name beam.autonomy in full through configengine's key-name helper, as PATTERN-config-key-names requires?"
  - "internal/beamcli/dobbydeps.go (tellRefusal, runAutonomyStamps) and internal/hubgeom/autonomy.go (AutonomyWithRefusal): trace a refused read, a successful read, a prime or chain failure, a failed post and two concurrent first refusals of one run. Which of them open, keep or end an episode, is the lock held across the creel post, and do the doc comments state what the code does?"
  - "internal/beamcli/dobbynoticestamp_integration_test.go: does a test pin that a read that succeeds ends a refusal episode and that a later refusal of the same run is posted again, on the same deps?"
  - "internal/loomcli/autonomy_integration_test.go, internal/loomcli/pawlwiring.go and internal/slackcli/wiring.go: for each reader that asks only whether autonomy is on, would a test fail if it compared against the `true` level alone and so read `unattended` as off?"
  - "_lym/landing/summary.md, section `## Merge danger`: do the door and the blast radius cover what an older build does with a beam config layer or a task's run_settings that holds `unattended`?"
  - "internal/orchcli/notify.go (WithRunAutonomy), with its callers in internal/battencli/notices.go, internal/loomcli/drivewiring.go and internal/loomcli/prepush.go: when a run's autonomy value is refused and only these halt notices are posted, what does the orch learn, and does the documentation in the code match that behaviour and the answer key?"
---

The entries cover the places the open findings touch, each phrased as a question about the tree as it stands.
