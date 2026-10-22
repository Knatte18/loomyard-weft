---
round: 1
focus:
  - "internal/tenderengine/resume.go, internal/pauserecord/record.go, internal/tendercli/deps.go: both sides edited the resume path and the pause record. Settle whether the task's owner check on resume still runs against the parent's new pause-holds-prompts behavior (PATTERN-pause-holds-prompts.md), and whether the record fields the parent added are honored by the task's owner-seam code."
  - "internal/loomcli/landingdeps.go, internal/landingshed/deps.go, internal/landingshed/stuck.go, internal/battencli/wire.go: both sides changed landing dependency wiring. Settle whether the task's owner seams (nil-legal) and the parent's changed deps still agree on signatures and defaults, and whether any seam is wired on one side only."
  - "internal/loomcli/ownercheck.go and the parent's loomcli changes (decision.go, darnwiring.go, reviewstatus.go, cli.go): settle whether a run owned elsewhere is still refused or skipped on every entry point the parent added or reshaped."
  - "PATTERN.md and the changed pattern/ files (glyph-conversion-chokepoint, pause-holds-prompts, review-round, verified-tree): settle whether the task's new code and PATTERN text follow each parent-changed invariant, and whether the task's PATTERN additions contradict the parent's wording."
  - "internal/testkit/envkit/envkit.go, internal/shedrun/paths.go and the overlapping *_test.go files (tenderengine resume_test/pause_test, pauserecord record_test, tendercli cli_test, landingshed mergein_test): settle whether tests from both sides now cover the same behavior, and whether any interaction (owner seam plus pause hold) has no test."
---

Seed focus from the overlap list and the two sides' diffstats.
Entries name areas and questions only.
