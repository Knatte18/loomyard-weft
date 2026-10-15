---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: decision
    location: tools/forkaudit/extract.jq:10
    summary: This run's rule that a task adding a pack layout adds its forkaudit marker row in the same task contradicts kickstart-per-card's recorded decision to leave that work to a follow-up task
    fix: >-
      The parent decides which record holds.
      Either kickstart-per-card's decision record moves the per-card marker row (prompt check before the file row, trace value card) into its own scope, and its Scope Out bullet and its Open risk on per-card forks classifying as file are amended to match;
      or this run's Constraints line and the rule sentence in tools/forkaudit/extract.jq:10 and the forkaudit.sh header comment are reworded to allow a follow-up task, the Open risk on kickstart-per-card is corrected to say per-card forks class as file until that follow-up lands, and the orch files that follow-up with depends_on glyph-audit-tool and kickstart-per-card.
---

## Scope check against the board entry and the decision record

The diff (PR 640 at `cf4ae4404`, merge base `c9c3fc97a` equal to `main`) carries every goal point of `glyph-audit-tool`:

- One tool under `tools/forkaudit/`, shell and jq only: `forkaudit.sh` with `extract.jq`, `pack-entries.jq`, `touched-files.jq`, `plan-writer.jq` and `report.jq` (D1).
- `--since`/`--until` and `--runs` input with self-discovery from the hub's encoded project directories (D2).
- Arm classification from the marker table at the head of `extract.jq`, with the trace cross-check and the `trace:<value>` rule (D3).
- The per-fork record and per-arm report with medians, pooled values and sample sizes (D4, D5).
- `--pack-check` and `--plan` as optional sections (D6, D7).
- Go integration tests on fixtures with the D8 `TestMain` shape, and the `tools/forkaudit: 5` row in `cmd/lyx/testdata/test-budget.yaml`.
- The `PATTERN-lyxdirs-single-declarer` line reads exactly the wording the parent's added decision prescribes.
- `docs/research/glyph-audit-2026-10-13/` and `docs/research/glyph-audit-2026-10-15/` are not in the diff.

Nothing in the diff goes beyond the goal.

Plan-Review's two carried-over findings are fixed in the code:

- `filters-spell-directory-roots-outside-variable-block`: no `.jq` file spells `_lyx` or `.lyx`.
  The variable block at `forkaudit.sh:108` spells each literal once, derives `RUN_RECORD_ROOT` and `LOGS_ROOT` by parameter expansion, and passes them and `PLAN_DIR` to the filters with `--arg`.
- `appended-records-path-breaks-identical-report-assertion`: the run-twice row passes one explicit `--records` path, so `runForkaudit` adds none and both runs carry the same arguments.

## Overlaps with runs in flight

`lyx loom peers` lists no parent commits; own row 22.

- `kickstart-per-card` (row 7, paused at Plan-Write, no changed paths yet): design conflict, finding F1.
  This run lands first by the landing order, and `kickstart-per-card` takes its changes at Merge-In.
- `role-language-skills` (row 14): it shares only the bookkeeping file `cmd/lyx/testdata/test-budget.yaml`, which is not an overlap.
  This run lands first by the landing order.

## Rejected

- The trace value `card` that `kickstart-per-card` adds is not itself a conflict: this run's D3 treats an unlisted trace value as a cross-check disagreement on a prompt-classified fork, which `kickstart-per-card`'s own Open risks expects.

### [BLOCKING] Who adds the per-card marker row: this task's rule against kickstart-per-card's follow-up

**Location:** `tools/forkaudit/extract.jq:10`, the `forkaudit.sh` header comment ("A task that adds a Webster pack layout adds its marker row there in the same task"), and this run's decision record, Constraints and Open risks.

**Issue:** This run's Constraints state that a task adding a pack layout adds its prompt marker row to `tools/forkaudit` in the same task, and the diff writes that rule into `extract.jq` and the script header.
`kickstart-per-card`, which adds the per-card layout and lands after this run, records the opposite in its Scope Out: forkaudit learning the per-card layout is a follow-up board task, filed by the orch, that depends on both tasks.
That follow-up is not on the board yet.
This run's Open risk also says per-card forks are "their own arm through D3's trace rule".
But the per-card prompt ends with a `## Start here` section, the `file` arm's marker, so those forks class as `file` with a trace disagreement, as `kickstart-per-card`'s own Open risks says.
Until one record gives way, the tool cannot measure the per-card layout against the file layout, and that comparison is this task's stated goal.
The parent-review alignment checked this rule against `kickstart-per-card`'s board entry, before that task recorded its decisions, so the contradiction is new since then.
