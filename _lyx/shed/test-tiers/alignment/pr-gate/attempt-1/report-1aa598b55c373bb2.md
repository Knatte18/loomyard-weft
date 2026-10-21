---
verdict: APPROVED
---

## Alignment at PR-Gate: `test-tiers`, PR 810 at `e45d65d7` against `main`

The diff matches the board entry's goal and the decision record's D1 to D13, including the operator's post-Discussion rule that zero discovered tests is not a failure.
The change spans 219 files, and main's tip `c2d13e5a` is merged in.

### Goal and decisions

- D1: `internal/gateslot/template.yaml` declares no tier, the `gates` lists D1 names, `scoper: none`, `guard_tier: 0` and an empty `refuse_raw`, and retires `go_parallel`.
  `yamlengine`'s set gains the flow-mapping `--set`.
- D2: `--tags` survives only as a refused flag that names `--tier` (`internal/gatecli/cli.go:96`).
- D3 and D4: the scoper lives in `internal/gatecli/scope.go`, with `internal/impactset` changed to emit per-module targets, and card scope reads the plan through `internal/planparser/targetdirs.go`.
- D5: the guard step lives in `internal/websterengine/guardstep.go` and `gatecli`.
- D6: `internal/gateslot/pool.go` adds the drain lock and ordered multi-slot acquisition.
- D7: `internal/gatecli/counters.go` adds the counters.
  The code has no minimum-test floor, as the operator's post-Discussion decision requires, so the board body's older "minimum count" and "zero is a failure" lines are rightly not built.
- D8: every listed site renders its gate's tier list.
  The `lyx gate card-verify` verb exists, the card runner moves to `internal/cardverify`, and the implementer stencil runs it.
  `verifytree.Verify` no longer acquires a slot.
- D9: `publish_verify`, darn's `verify` and `verify_tiers` are retired, and `tree_scope_verify` is validated as one `lyx gate test --wait` call with no shell control operator (`internal/darnengine/config.go:297`).
- D10: the shell guard gains a refuse-raw row, and `claudeengine` renders it into settings.
- D11: `internal/gateslot/runlog.go` writes the per-run log and prunes it after 14 days.
- D12: the `CLAUDE.md` push rule now reads `lyx gate test --tier 0 --tier 1 --tier 2` on `main`, and the loomyard example sits in `internal/gatecli/doc.go`.
  Setting each hub's tier values is the orch's job after the deploy, as D12 says.
- D13: `cmd/testtiming` holds a slot in every mode, and `internal/planglyph`'s type load keeps its slot with its own GOFLAGS.
- The PATTERN changes match the record: `PATTERN-gate-slot` is rewritten, and `PATTERN-test-tiers` is added with background files.

### PATTERN overview trims

To keep the overview under its headroom line, the run shortens three unrelated overview lines: `PATTERN-push-both-sides`, `PATTERN-recipe-setting-lookup` and `PATTERN-sparse-hub-base`.
Each dropped clause is still in that entry's background file.
These are wording-only cuts forced by the two added links, and no invariant changes.
They are bookkeeping, not a scope deviation.

### Carried-over open risks

- Plan-Review round 4 closed on budget with `absent-gate-yaml-refused-at-wiring`, `decodeconfig-resign-arrow-missing` and `shared-fixtures-missed-by-new-behavior` open.
  Its fixer report applied F1 to F3 and deferred only a self-check fixture finding, which Webster's run then had to resolve to pass its gates.
  These are design details, not scope.
- Merge-Review round 1's `gates-open-map-single-set-drops-other-lists` and `darn-fixture-retired-verify-substring-accident` are addressed by `5935d624`, `372cbc98` and `976e1b4a`.
- Webster-Review's `triage-section-names-retired-verify-suite` is a MEDIUM consistency item, not scope.
- The record's risk on `tuned-defaults-template` landing order is settled.
  That task landed first, as `c2d13e5a`, and is merged in, and `5935d624` keeps `gates` a per-name map so a hub's single gate `--set` keeps the other lists.
- `origin-reconcile` landed as `e610f925` and is merged in.

### Overlap and landing order

`lyx loom peers` lists no peer run and no unmerged parent commit (`own_row: 21`), so no overlap needs a landing order.
Related board entries are consistent with this run:

- `tender-pause-holds-reprompt` keeps pause semantics, which the record puts out of scope.
- `darn-verify-fixes` is re-scoped after this lands, per the open risks.
- `dotnet-verify-protocol` keeps Wanted 4 to 6 and its rubric rules.
- `regression-baseline-verify`, `dotnet-scoped-verify` and `gate-status` stay out of scope.

None of these is made obsolete, so `superseded` is empty.
