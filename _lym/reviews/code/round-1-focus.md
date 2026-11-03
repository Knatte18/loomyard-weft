---
round: 1
focus:
  - "internal/driftcli/wiring.go, internal/loomcli/driftwiring.go, internal/driftengine/spawn.go, capture.go, compare.go: does the capture child, the compare child and the capture and compare commands start in the anchored directory, resolved through internal/lymcwd from .lym-anchor and passed as an absolute path, with no hand-joined anchor path anywhere in the drift producer (decision record, Added after Discussion, and PATTERN-cwd-resolution)?"
  - "internal/driftengine/deps.go, seats.go, verbs.go, steprun.go: with drift.enabled=false in the local layer, does a resume pass Drift-Baseline and Drift-Verify through, including a row the run already entered, and is drift.enabled read at every pass? Check the cases where the flag flips between passes or while a child is running."
  - "internal/driftengine/verbs.go, baseline.go, await.go, verify.go: is lym drift defer admitted at a failed capture halt, and does that halt name it in its way forward? Check that a capture child that never records its start maps to the capture halt and that defer settles it without leaving stale state."
  - "internal/driftengine/verbs.go, internal/driftcli/cli.go: the parent can record or replace the scope at a blocked Drift-Baseline or Drift-Await. Check which statuses admit a second lym drift scope, that a replaced scope invalidates anything derived from the earlier one, and that the run's own agent still cannot overwrite a recorded scope."
  - "contracts/stencils/drift/drift-template-baseline.md and contracts/stencils/markers_test.go: does the stencil tell the agent to check whether the capture command's projects reach the changed files through project references, and to record scope off when they cannot or when the change touches no code the suite runs? Check that no rule the card did not ask to change is dropped, weakened or contradicted, and that the capture command the prompt names is supplied by the code that renders it."
  - "Test economy across internal/driftengine/*_test.go: a new test duplicating an existing test, or a behavior the diff adds with no coverage (the anchored start directory, the enabled=false pass-through, defer at a capture halt, scope replacement)."
  - "docs/overview.md, internal/driftengine/doc.go, internal/driftcli/doc.go, internal/shedrecipe/doc.go, entries_drift.go: do the doc and recipe changes match the behavior the diff makes, and does the commit message claim anything the diff does not bear out?"
---

Seed focus for round 1.
The measuring stick is the decision record: its board entry under `## Goal` (three faults, #23 anchor start directory, #24 skipping drift after a failed Drift-Baseline, #25 scope off for unreachable changes) and the parent's amendment on how the anchored directory is resolved.
The summary file is absent, so the commit message is the only account of the diff.
