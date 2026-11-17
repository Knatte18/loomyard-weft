---
round: 1
focus:
  - "internal/heddleengine/shipped.json: do the three reworded lines keep each rule's meaning, and does any line still name a Claude-only tool (Read, Edit, Write, grep as a tool) that a Codex session would not hold?"
  - "internal/heddleengine/render_test.go: does the new TestRenderRemovesAStaleMarkedRenderEvenOffExcluded, which replaces shippedStore and is not parallel, restore the shipped layer afterwards, and do the parallel subtests of TestRender stay safe against that replacement?"
  - "internal/heddleengine/render_test.go: does fileTargetWant mirror the production render's rule for which shipped entries reach each file, or does it re-derive the expectation so a render bug would pass unnoticed?"
  - "internal/heddleengine/seam_integration_test.go: does the prefix and suffix check on AGENTS.override.md still pin the render's content and order, or has it weakened the earlier exact comparison?"
  - "Test economy: do BothFilesHoldTheShippedLayerWithNoEntry's heading assertions duplicate the exact-equality check beside them, and does the moved stale-render test cover anything TestRender's other cases do not?"
  - "Scope: the answer key asks to check that a rendered AGENTS.override.md holds both entries; does the diff show that check, and does it change anything beyond shipped.json and the tests that pin the rendered files?"
  - "overview_test.go: does moving the 'From: shipped with lym' assertion from the claude-file section to the agents-file section match what the overview now renders for the shipped entries?"
---

Seed focus for round 1.
The diff is one commit touching `shipped.json` and four test files; the decision record holds only the board entry under `## Goal`.
