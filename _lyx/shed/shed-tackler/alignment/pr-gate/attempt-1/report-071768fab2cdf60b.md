---
verdict: APPROVED
---

## Alignment at PR-Gate: shed-tackler, PR 636 at 9fbabe149

The diff against `main` (merge base `6667f79df`, which is `main`'s tip) matches the board entry's goal and the decisions D1 to D9 of the decision record.

### Goal against the change

- The Go driver and the whole tackler moved out of `internal/loomcli` and `internal/loomengine` into the new `internal/sheddriver` (D1).
  Its production imports exclude `lyxcwd`, `loomengine`, `loomshed` and every `<module>cli`, and `internal/sheddriver/seam_enforcement_test.go` pins them.
- Loom and darn are driven through the unchanged hidden `lyx loom drive` verb, and `cmd/lyx` gains no `lyx shed` verb (D3).
- Batten drives its own run through the `sheddriver` body in `internal/battencli/drive.go`, with no arbiter, hand-over or tamper seam and `GotoRoutedFrom: false` (D2, D4).
- The new `shed` config module `shed.yaml` is registered in `configreg` with today's defaults.
  `loom.yaml` and `darn.yaml` carry four inheriting overrides, and `batten.yaml` carries three tackler overrides with no `driver_kind`.
  Loom's `Migrate` uses the two-step value-and-comment migration that D5 states (D5).
- Both seeding sites resolve `driver_kind` from the child recipe's own override, then `shed.yaml` (D6).
- The tackler stencil now lives at `contracts/stencils/shed/shed-template-tackler.md` and carries the `{{.recipe}}` and `{{.goto_rows}}` markers, a stop-kind list and a reading order.
  It does not name the arbiter (D7).
- `PATTERN-shed-driver` is added with a background file, and `PATTERN-told-geometry`, `PATTERN-shed-verb-set` and `PATTERN-driver-choice-single-site` are updated as D1, D4 and D6 say (D9).
- The out-of-scope items are respected:
  - every `driver_kind` default stays `llm`;
  - the arbiter stays loom's;
  - no run-verb namespace or message receiver is added.

### Open risks carried in the record

The batten tackler's write rule is enforced only by the stencil, and a killed batten step child becomes an `interrupted` stop.
Batten's notice episodes, go-driven runs having no message receiver, `twb-d0-sa`'s darn seams and the tackler's first live runs are also open.
All of these are recorded decisions or accepted residuals, not deviations.
Discussion-Review's carried finding `migration-comments-stale-on-pinned-key-hubs` was addressed in D5's comment-refresh step, and the change implements that step in `internal/loomengine/config.go` (`migrateShedOverrides`).

### Note on the PATTERN index

Card 10 shortens five existing PATTERN index lines so the overview stays under `HeadroomBytes`:

- `PATTERN-no-denied-recovery`;
- `PATTERN-push-both-sides`;
- `PATTERN-verified-tree`;
- `PATTERN-agent-name`;
- `PATTERN-test-speed`.

The plan names three of these and allows further cuts of the same kind.
Each dropped clause is still stated in its background file, so no invariant changes.

### Overlap

`lyx loom peers` lists no peer runs in flight and no landed parent commits (`own_row` 22), so this run overlaps nothing and no landing order applies.
The shared bookkeeping files `cmd/lyx/testdata/test-budget.yaml` (new `internal/sheddriver` row), the module table in `docs/overview.md` and the new PATTERN index line change, and none of them escalates.

### Board

`run-pane-timing` and `refusal-route-config-empty` depend on this task, and the change does not touch their scope.
`orch-split`, `run-verbs-shared` and `loom-model-key-names` stay open and are not made obsolete: the record leaves their work out of scope.
