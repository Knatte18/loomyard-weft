---
round: 1
focus:
  - "internal/landingshed/prreview.go, deps.go, internal/loomcli/landingdeps.go: with the setting on, does PR-Review ask the parent orch once the PR is open, naming slug, PR URL and branch, and wait until the orch answers satisfied or sends findings? Check the failure paths: no parent, post refused, repeated entry into the step."
  - "Same files: with the setting off, is PR-Review behavior unchanged from before the diff?"
  - "internal/loomengine/config.go, template.yaml, config_test.go: is `pr_review_ask_parent` a loom key with a per-task `run_settings` override, defaulted off, resolved per hub and per task like the PR-Review perch's other keys?"
  - "contracts/orchrole/orch-role.md and landing-template-pr-review.md: do they instruct the orch how to answer such a request, and the reviewer how to handle the orch's findings as in the by-hand flow? Check that no existing stencil rule is dropped or weakened."
  - "internal/pullrequest/record.go and internal/testkit/envkit/envkit.go: is each change licensed by the board entry, or is it scope beyond it?"
  - "internal/battencli/wire_seams_test.go and internal/loomcli/wiring_seams_test.go: the diffs are large against the behavior added. Are the rewrites needed, and does every new behavior (landingshed prreview.go) have coverage without duplicating prreview_session_test.go or prreview_test.go?"
  - "docs/overview.md and landingshed/doc.go: do the docs the repository's constraints name for this kind of change land, and does each claim match the code?"
  - "Commit messages: any Pattern-Departure trailer, and whether it names a real entry that forbids the answer key's way."
---

The decision record holds only the board entry under `## Goal`, so the entry is the measuring stick.
