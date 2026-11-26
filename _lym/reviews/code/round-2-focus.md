---
round: 2
focus:
  - "internal/landingshed/prreview.go, openPullRequest, wake, judge, gateHead and foreignHeadHalt: on every path, a re-entry after a gate-refusal or foreign-head halt included, can a task head reach the pull request branch before the pr-review gate passed it, or can a hand commit on GitHub end in a push halt instead of the foreign-head halt? Do the halt texts, doc comments and docs/overview.md match the order the code runs in?"
  - "internal/landingshed/prreview_wait_test.go and the PR-Review Call tests: do the tests fail if foreignHeadHalt moves after the continue or approval ending, or if the entry push runs before the gate on a re-entry?"
  - "The upgrade from beam.pull_request on or merge: trace every place a stored value lives (hub beam.yaml, worktree local layers, board entry run_settings) on a machine whose beam-per-task-settings carry marker already exists. Is each rewritten or flagged, and does configreg.PrepareRunSettings accept the result? See internal/dobbyengine/reinterpret.go and internal/hubreconcile/reinterpret.go."
  - "The text of the on to review rewrite and flag in internal/dobbyengine/reinterpret.go: does it state what differs for a hub with beam.autonomy true?"
  - "The retired pr-reviewer role: does a hub agents.yaml holding skills.pr-reviewer or a pr-reviewer/<model> efforts key still load through agentconfig.Load after the upgrade walk, with other keys and comments kept?"
  - "_lym/landing/summary.md, Merge danger: does the blast radius name everything the diff can break for another binary, config, records and removed verbs included?"
  - "Any new migration code that writes board entries or hub config files: is it safe to run twice, after a partial failure, and on a hub whose machines run different builds?"
---
