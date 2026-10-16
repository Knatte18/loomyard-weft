---
round: 2
focus:
  - "In D4 of _lyx/discussion/decision-record.md, batten drops the tamper snapshot of the prime. Settle whether the Decision states what a batten tackler that writes in the prime (files, index, a local commit) can now let through, and what mechanically bounds it, given that a batten resume may re-enter Land or Worktree-Teardown and that the stop-key check compares run status, not writes. Check the D4 Bound line and the acceptance criteria name the same bound."
  - "In D5 of _lyx/discussion/decision-record.md, loom's Migrate hook rewrites values equal to the old defaults. Settle whether the Decision states its scope and bound given that configreg.Module.Migrate runs on every reconcile and, via internal/configreg/runsettings.go, on every task's run_settings document, and whether the shipped templates of loom.yaml, darn.yaml and batten.yaml carry the inherit value so a fresh hub does not mask shed.yaml. Check the Migrate acceptance criterion covers the chosen scope."
  - "Across D3, D5 and D6 of _lyx/discussion/decision-record.md, settle which module's override (loom.yaml, darn.yaml, batten.yaml) a run served by lyx loom drive reads for each tackler key and for driver_kind, chosen by the seed's recipe, and whether every key added to darn.yaml and batten.yaml has a reader."
  - "In D2 and D4 of _lyx/discussion/decision-record.md, settle where batten's halted-row-only resume goto is enforced when today's verdict validation in internal/loomcli/tackler.go accepts the current row and the routed-from row, and whether the acceptance criteria check a refused routed-from goto."
---

Round 2 targets the open design questions around bounds on removed guards, the Migrate scope and per-recipe config resolution.
