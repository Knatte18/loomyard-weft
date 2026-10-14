---
verdict: APPROVED
---

## Alignment at PR-Gate: `perch-budget-arbiter`, PR 621 at `75dcf69d3`

The diff matches the board entry's goal and the decision record as its "Added after Discussion" entries amend it.

- One budget shape: `internal/reviewbudget` declares the key family, categories, cap resolution, layer routes and bound checks; `loomengine` and `darnengine` build their `Categories` and checks through it, and `pattern/PATTERN-leaf-packages.md` gains its row (D1).
- Darn per-run budget: darn's `review_max_bounces` and `review_hard_max_bounces` lose `CategoryGuard`, `configreg` gains darn's `CheckValues` hook, and darn refusals name the way forward for the value's layer (D2, "Perch budgets carry no guard").
- Zero-round perches: `skip_review`, the darn migration and the `PrepareRunSettings` reorder are absent, as the parent's "Zero-round perches: settled design" entry drops them.
  The per-perch loom budgets are `*int` with a null template value; 0 means zero rounds; `skip_merge_review` is retired, and `MigrateConfig` rewrites it onto `merge_max_bounces: 0`, with old per-perch zeros becoming null.
  A loom perch at 0 keeps its writer → Bouncer route, and a stamped `zero_rounds` Bouncer settles approved through its seams ("Zero-round perch: the Bouncer settles at once").
  Darn at 0 keeps Darn-Write → Merge-In.
- Arbiter: `internal/arbiterstop.Eligibility` is the one rule the verb, the Go driver and batten share; an awaiting stop is eligible only at a `hard-cap` or `circling` cause, and an unreadable record or unknown cause is a refusal with the hand-back way forward (D5).
  The `blocked` and error rules are unchanged (D6), and the arbiter stencil drops its `budget` rule.
- Driver stencil: a twice-failed transient goes to the fork first (D7), the verb decides eligibility, an awaiting refusal is a planned hand-off (D8), and no notice is relayed after a `resume` (D9).
- Batten hold: `battenshed/stophold.go` holds an eligible blocked or awaiting stop's notice until the park marker, a hand-on read from `envelope.json`, an applied mark, the arbiter turned off, or the backstop; it then measures the report wait from the hold's end (D9).
- `yamlengine --set` tagging over a null leaf addresses the Plan-Review carry-over `null-per-perch-budget-breaks-config-set`.
  The hand-on end time from `envelope.json`'s write time matches D9 as written, which the Discussion-Review carry-over `d9-hand-on-end-time-file` concerns.

The record's acceptance criteria still list `skip_review`, its migration and the refusal of `{"darn":{"review_max_bounces":0}}`.
The parent's settled-design entry withdraws those criteria, so their absence is no deviation.

No change falls outside the goal: the `yamlengine` change serves the null-inherit budget through `lyx config --set`, and the `arbiterstop` package exists because `battencli` cannot import `loomcli`.

## Overlap

- `overview-seats` (row 12, `blocked · Webster`): both runs change `contracts/specs/refusal-spec.md`, `docs/overview.md`'s loom config paragraph, `internal/loomcli/doc.go`, `internal/loomcli/wiring.go` with its darn and loom wiring tests, `internal/loomrecipe/loomrecipe.go` and `doc.go`, `internal/darnengine/doc.go`, `internal/shedadapters/doc.go`, `internal/battencli/wire_seams_test.go` and `internal/testkit/envkit/envkit.go`.
  Its D11 also extends `loomengine.MigrateConfig` and `ConfigSpec`, where this run adds the `skip_merge_review` retirement and the `reviewbudget` categories.
  The changes touch different keys and behaviour (`discussion_producer`, `plan_producer` and advisors there; review budgets and `skip_merge_review` here), and its checklist reads arbiter attempt envelopes this run leaves unchanged, so the overlap is in text, not design.
  Landing order: this run (`own_row` 22) lands first, and `overview-seats` takes its changes at Merge-In.
- `twill-fork0` (row 7) and `delete-order-own-edit` (row 0, `internal/planglyph` only): no overlap.
- Parent commits: none listed.
- Shared bookkeeping: `cmd/lyx/testdata/test-budget.yaml` and the new `PATTERN-leaf-packages.md` row.
