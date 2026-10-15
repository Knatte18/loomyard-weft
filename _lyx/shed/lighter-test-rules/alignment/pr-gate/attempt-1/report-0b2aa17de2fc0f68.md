---
verdict: APPROVED
---

The diff of pull request 654 at `3394b072d` against `main` matches the board entry's goal and the decision record.

- The board's later operator decision (drop wording claims entirely; keep marker sets, Fill round-trips, PATTERN scans and registry tests) is what D1 encodes, and the diff applies it: `contracts/stencils/claims_test.go` is deleted, and the template and prompt tests under `internal/` and `cmd/lyx` lose their static-prose assertions while keeping Go-filled values, Go-chosen branches and marker checks.
- D2's placements are built: `contracts/stencils/govalues_test.go` holds the Go-value table (`gatecli.SlotBusyExit`, `shuttleengine.MessageTail`, `shedrun.ParkMarkerFileName`, the `websterengine.Report` yaml keys) and the twill host-neutral scan, both `//lyx:guard`;
  `markers_test.go` gains the marker-position table with the kick-start pack, inline pack and section-scoped rows, and `TestStencils_TopLevelMarkers` gains rows for stencils no other test reads in their real file, the darn Describe stencil among them.
- D3's rule text lands in `contracts/specs/loom-plan-spec.md`, the plan stencil, `pattern/PATTERN-test-economy.md` and the plan-review, webster-review, darn-review and merge-review rubrics; the `PATTERN.md` one-liner is unchanged as D3 says.
- D4's **Stencil wording** bullet lands in the webster-review rubric (measured against the card's `Intent`) and the darn-review rubric (measured against the board entry).
- D5 holds: `cmd/lyx/testdata/test-budget.yaml` is not in the diff.
- Nothing in the diff falls outside the record's In list; the doc rule, `CLAUDE.md` and Go-built text assertions are untouched, as the Out list says.
- The only remaining mention of `claims_test.go` is in `docs/research/glyph-audit-2026-10-15/control-check.tsv`, a measurement record.

The Discussion-Review carry-over (`acceptance-criterion-2-contradicts-d1-keeps`, `d4-bound-overstates-coverage`, `marker-placement-between-d1-d2`) was addressed by the round-3 fixer per `_lyx/reviews/discussion/round-3-fixer-report.md`, and the decision record carries those fixes; the diff is consistent with the fixed D2 and D4.
The record's open risk naming `engine-registry` and `creel-core` no longer applies: neither is among the runs in flight.

## Overlaps

`lyx loom peers` lists no parent commits; `main` has no commit past the merge base `993ff3fc5`.

- `engine-config-seams` (row 9, Plan-Burler-Review): no overlap.
  Its scope is engine-owned config, the shell-guard table and provider-blind reed; it touches neither stencil test rules nor the files this diff changes, and has no changed paths yet.
- `engine-model-choice` (row 7, Plan-Write): no design overlap.
  It may touch carriers in `internal/burlerengine`, `internal/websterengine` and `internal/shedadapters`, packages whose test files this diff prunes; that would be a text-only overlap at different places.
  If it adds a static-prose assertion, the record's second open risk covers it.
- Landing order: this run (own row 22) is furthest along, so it lands first and both peers take its changes at Merge-In.

## Superseded

No board entry is made obsolete by this run.
`discussion-relevant-list` depends on it and stays.
