No findings beyond one stale twin.

- LOW, `internal/orchengine/doc.go:433`: describes the wake line as "telling the orch to run `lym tender revive` and check each run".
  The rewritten stencil tells the orch to wait for notices of the runs that stopped, so this doc comment states the old behaviour.
  Evidence: that line against `contracts/stencils/orch/orch-template-usage-limit-wake.md:4`.

Checked and consistent: the wake line against role rule 4 (`orch-role.md:55-56`), the Status reports sentence against the "Never `cd`" rule, and the registry test's `orch-template-role` name against the fixed stencil key `orch-role`.

Tool calls: one, a grep for "check each run" and "batten status <slug>" across Go and Markdown outside `docs/research`, because the context lacked other twins of the wake line.
