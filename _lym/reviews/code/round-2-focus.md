---
round: 2
focus:
  - "contracts/stencils/darn/darn-fragment-make-change-gated.md: does the gated writer, for a reported test failure, ask its parent through creel for one run of that test before diagnosing, without naming a package, target, scope form or test-command argument (PATTERN-test-speed)?"
  - "The same fragment: does the request name the test exactly and its tier, and does the writer read a pass as the failure no longer reproducing only when the report shows that test ran, not when it was skipped or matched nothing?"
  - "The same fragment: does the reproduce instruction come before the writer's first edit or diagnosis, and does item 3 again say the writer runs no `lym gate test` itself?"
  - "contracts/stencils/stencils.go, internal/darnengine/spec.go and internal/darnengine/doc.go: do the comments describing the gate exception match the parent-route wording, with no leftover claim that the writer or its subagents run the test?"
  - "internal/darnengine/spec_test.go and internal/loomcli/wiring_darn_test.go: do the tests assert only which fragment Go chooses, with no assertion on the fragment's static prose?"
---

The focus follows the commits in the review range that change the gated fragment and its tests.
