---
round: 1
focus:
  - "internal/adoptcli/worker.go, steprun.go, steps.go: does resume follow D2 (start at reconcile when the first step not done is owner or earlier, keep done steps and inputs, flag replacement for --keep-local/--drop-local) and does a lost lock exit writing nothing?"
  - "internal/adoptcli/steprun.go: does a failed step store its gathered part (the parent's Added-after-Discussion entry), clear it on later success, and merge it after the done steps in the end envelope without listing the step as done?"
  - "internal/adoptcli/caller.go, follow.go: pid and start-ticks ownership, follow of another pid's record only when its start is later, exit codes for done, stopped and interrupted, the keep-warm cap with 'still running', and the wake backoff."
  - "internal/adoptcli/pause.go: D5 rules — OwnerCaller entries converted in place keeping pause time, a slug holding another caller's entry gets none, slugs the record already lists as paused are skipped, beam pair slug conversion, unstarted non-front released tasks."
  - "internal/adoptcli/green.go, holds.go, runreport.go, and the pauserecord/dobbyengine/battencli/landingshed edits: the green end only reports (D8); the adopt pause and legacy adopt-hold are lifted only by hand, never by the check, the owner lift or a hibernate wake; holds.json is neither written nor read; paused_by_adopt replaces the two old states."
  - "internal/orchcli/start.go and the caller's --no-orch default from an orch strand via agentname.Parse (D7); internal/beamcli/adoptsection.go and cli.go for the D9 adopt section (unfinished states only, way forward per state)."
  - "Scope against the plan: D4 is absent from the record, so check for unlicensed additions; check the diff against docs/overview.md, pattern/ changes (PATTERN-dobby-runs declared text, PATTERN-daemon-wakeups), doc.go claims, and the repo's boundary rules (git only in fabricengine, never-tracked files under AdoptScratchDir)."
  - "Test economy: TestAdoptFields (contained, covered by TestAdopt) and TestTriesStore (contained, covered by TestFailedEnd) — compare assertions before filing; also the new adoptrecord, beamcli and hibernateengine tests for coverage gaps and duplicated coverage."
---

Focus covers the worker, record, follow, pause and green-end changes, which carry the decision record's behavioral criteria.
