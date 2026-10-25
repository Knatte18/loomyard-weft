---
round: 1
focus:
  - "Capture lock and compare marker (D4): check that probes never make a capture refuse, a released lock is the only liveness signal, and `agentActivityFor` reads both so the idle watchdog stays off during a capture, a queued compare and `Drift-Await`'s wait."
  - "Windows breakaway and teardown (D4): check that `JOB_OBJECT_LIMIT_BREAKAWAY_OK` is set on both the `proc.StartTied` job and the loop's named job, that the capture holds its named job through `proc.HoldNamedJob`, and that `pairteardown` terminates it through `proc.TerminateNamedJob`, the local adopt's end included, with the foreground fallback when breakaway is refused."
  - "`Drift-Baseline` and `Drift-Await` state machine (D3, D5): check the re-entry cases against the decision record, namely scope present without `baseline.md`, `awaited_at` set, a running capture whose lock is free, a HEAD that moved or a dirty tree, and that every Stuck names the right way forward and carries the capture-halt marker that `lyx drift defer` reads."
  - "`Drift-Verify` and the scope guards (D6, D7): check that under scope `full` every halt but an error or transient stop is a decision halt for the parent, that `lyx drift accept`, `waive` and `defer` refuse outside their admitted halts, and that a loom Finalize reroute compares again while darn's Code perch fixes and a Finalize merge go uncompared."
  - "Config and off switch (D1, D2): check that `drift` is `HubWide` with executable keys read only from the template and board default layer, that a `capture` or `compare` command without `{baseline_dir}` is refused at load, that `enabled` true with an undeclared command refuses naming `lyx config drift`, and that all three rows carry `entry_gate: drift.enabled` so a hub with the flag off sees nothing happen."
  - "Recipe wiring and repository rules: check row positions in `contracts/recipes/loom-recipe.yaml` and `darn-recipe.yaml` against D2, the `drift-verify` gate key, and that `PATTERN.md`, `pattern/` and the `doc.go`, `docs/overview.md` and doc-list updates the task requires land in the same diff, with no change to an existing producer's behaviour, Discussion's stencil included."
  - "Test economy: check the new drift tests for duplicated coverage, per-helper tests the public surface already covers, and behaviour added with no test."
---

Seeded from the decision record's scope and decisions; the plan's lens files were not used as a measuring stick.
