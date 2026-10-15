---
round: 1
focus:
  - "internal/configreg/configreg.go: the parent added `Module.CheckLoad` for every module and the task added the `shed` module and batten's `CheckValues`; check each merged entry sets both its `CheckValues` and a `CheckLoad` that matches the loader the module's readers use, and that `sheddriver.LoadConfig` refuses exactly what a later reader of loom, darn and batten overrides refuses."
  - "internal/loomengine/template.yaml with the tackler keys now empty or null overrides: check every loader and writer path the parent's `CheckLoad` runs (config write, local-layer check) accepts the inherit values (`\"\"`, `0`, `~`) and resolves them through shed.yaml the same way for loom, darn and batten."
  - "contracts/stencils/claims_test.go, stencils.go and the stencil tables: the parent added twill and webster inline stencils and claim tables while the task moved the tackler stencil to `shed-template-tackler`; check the registry, `roleOpeningStencils` and the claim-table coverage and host-neutral checks still cover every entry with no stale or duplicate row, and that no remaining reference names `loom-template-tackler`."
  - "cmd/lyx/testdata/test-budget.yaml with internal/configreg tests: the parent's `CheckLoad` non-nil test and the task's `runsettings_test.go` additions; check the budget rows both sides edited agree with the merged top-level test counts, and that no merged test duplicates another."
  - "docs/overview.md and internal/loomcli/doc.go: both sides edited them; check the merged text describes the shed-layer driver and tackler move and the parent's changes without contradiction or stale package references."
---

Focus is drawn from the overlapping files; the verify gate already covers build and test failures.
