---
verdict: APPROVED
---

## Scope against the board entry

Every Scope point of `twill-fork0` has its decision, and nothing in the design reaches past them:

- Scope 1, the fork0 flow at Plan-Write: D1 to D5.
- Scope 2, the strict critical fixer: D6, modelled on `burler-step-fix.md` as the entry asks; the entry leaves Discussion-Write's share to Discussion, and D6 decides it (auto-pick Q1, recorded under Open risks).
- Scope 3, the `branch-coverage` lens: D7.
- Scope 4, the `forced-edits` registration sentence: D8.
- Scope 5, the wrong-parent-directory miss: D9, in the `criteria-coverage` lens, one of the two places the entry allows.

The entry's "Not built" list is honoured in the record's Out list: no rationale file, no Plan-Review round counts, no efficiency audit, none of the bench agent's untested ideas.
The supporting changes (the host-neutral fork boilerplate of D5, the retired `rejection_rule` host value, the host-neutral re-prompt rule of D3) follow from the scope points and add no behaviour the entry does not call for.

No other board entry is duplicated or contradicted.
`fan-master-preread` and `fan-prompt-tuning` own Burler's lens fans and fork prompts, which the record puts out of scope.
`lighter-test-rules` is cited as a constraint, not built.
`loom-model-key-names` is untouched: fork0 adds no model key.

## Overlaps

`lyx loom peers` lists no landed parent commit.

- `overview-seats` (row 12, Webster; own row 3).
  It retires `plan_producer` and `discussion_producer` for `plan_lenses` and `discussion_lenses`, moves Plan-Write and Discussion-Write onto seat tables with the twill directive in the lead seat alone, and changes `internal/loomengine/twill.go`, `plan.go`, `discussion.go` and their tests.
  This run's D0 builds on exactly those terms, so the designs agree; the overlap is the same twill hosts and tests changed at different layers.
  The board entry carries no `depends_on` for it; the support log's Q9 records that `ly:orch` holds this run before Plan-Write until `overview-seats` lands and merges main in, and the record's Auto-mode assumptions say the same.
  Landing order: `overview-seats` is further along and lands first; `twill-fork0` takes its changes at Merge-In.
- `perch-budget-arbiter` (row 12, Webster; own row 3).
  Shares `internal/loomengine/template.yaml` and `docs/overview.md`, where this run edits the `plan_lenses` comment and the twill lines; its decision record names no twill, lens or judge behaviour, so the overlap is text-only.
  Landing order: `perch-budget-arbiter` lands first; `twill-fork0` takes its changes at Merge-In.
