# Review budgets per run for darn and loom; arbiter only at needed stops

Darn and loom now share one review-budget config shape, any review perch can run zero rounds per run, and the arbiter spawns only where an opus judgement is needed.

## What changes

- **Shared budget keys.**
  The new leaf package `internal/reviewbudget` declares the budget key family once: run-wide `review_max_bounces` and `review_hard_max_bounces`, and the per-perch `<perch>_max_bounces`, `<perch>_hard_max_bounces` and `<perch>_auto_accept`.
  It owns their categories, value checks, refusal wording, the soft/hard cap resolution and the layer-aware way forward.
  `loomengine` and `darnengine` build their config from it, so the two recipes cannot drift apart.
  Its row is in `pattern/PATTERN-leaf-packages.md`.
- **Darn budget per run.**
  Darn's `review_max_bounces` and `review_hard_max_bounces` are ordinary keys, so a darn run takes them through board `run_settings` as a loom run does.
  Darn cap refusals name the way forward for the layer the value came from.
- **Zero-round perches.**
  Every perch in loom and darn takes a budget of 0, meaning no review round, set per run or hub-wide.
  A Bouncer whose resolved budget is 0 settles approved at once and still runs its approve and commit seams, so a plan is still approved and committed.
  Darn at 0 keeps the Darn-Write to Merge-In route and skips Describe.
  No perch budget is a guard key, so a task can switch off any review, Discussion-Review included, through its own `run_settings` or local layer.
- **Inherit is a null value.**
  A per-perch key that is null in the template inherits the run-wide value; 0 now always means zero rounds.
  Loom's `Migrate` hook rewrites an existing `<perch>_max_bounces: 0` to null.
  The hard-cap keys are unchanged: 0 still means inherit.
- **`skip_merge_review` is retired.**
  `merge_max_bounces: 0` replaces it, and `skip_merge_review: true` migrates to that.
  The Merge-Review route switch is gone from the loom recipe; Merge-In still runs into Merge-Bouncer and Publish.
- **Budget values refused at upsert.**
  A `run_settings` budget below its bound is now also refused at `lyx board upsert` for both recipes, where loom refused it only at load before.
- **`yamlengine --set`** gives a scalar the tag its own text resolves to when the text or the leaf is null, so a null key can be set to a number and back.
- **Arbiter eligibility.**
  The new `internal/arbiterstop` holds the one eligibility function, used by the verb, the Go driver and batten.
  An awaiting stop is eligible only at a pending `hard-cap` or `circling` escalation.
  A `budget` escalation, or an unreadable or unknown-cause record, is refused and goes to the parent.
  `blocked` and error eligibility are unchanged.
  `shedadapters` exports a reader of a pending escalation's cause; the arbiter stencil drops its `budget` rule.
- **Parent hears after the arbiter.**
  Batten holds the state-change notice of an arbiter-eligible stop until the arbiter hands on, a park marker is written, a resume is applied, the arbiter is switched off, or a backstop passes (stop time plus `arbiter_timeout_min` plus the report wait, or the live attempt's deadline plus the report wait).
  When the hold ends, `stateChangeReady` measures the report wait from the hold's end, so the notice does not go out with `report none yet` ahead of the driver's relay.
  Error stops keep only the existing marker hold.
- **Driver stencil.**
  A twice-failed transient goes to the repair fork before the arbiter.
  The verb decides eligibility, a refusal at an awaiting stop relays the `parent_notice` as a hand-off, and nothing is relayed after a `resume`.
  The orch stencil's description of darn's off switch follows.

Docs and specs are updated in the same change: `internal/loomcli`, `internal/darnengine`, `internal/loomrecipe` and `internal/battenshed` `doc.go`, `docs/overview.md`, and `contracts/specs/refusal-spec.md` rows for the new and reworded refusals.

## Design decisions to know

- Darn's `review_max_bounces: 0` is its off switch; a hub `darn.yaml` already holding 0 keeps its meaning with no migration.
- Zero-round perches stay routed through their Bouncer rather than past it, because Plan-Bouncer's approve seam is the only writer of `approved: true` and Webster refuses an unapproved plan.
- A `budget` escalation now always costs the parent a turn, since the soft-cap call belongs to the Bouncer's rule.
- The batten hold takes eligibility from the same function as the verb; `battenshed` holds no copy of the rule.

## To check

- Merge conflicts: the parallel `overview-seats` task edits `internal/loomcli/checklist.go`, `internal/loomcli/doc.go`, `contracts/specs/refusal-spec.md` and `configreg`; expect conflicts in whichever lands second.
- A darn soft cap above the hub's hard cap, set in a hub file or `run_settings`, passes `lyx board upsert` and is refused at load, parking the run until a cap is changed.
- Null per-perch budget and `config set`: a per-perch budget whose template value is null may break setting it through `lyx config set`; check that path by hand.
- Batten's hold was only exercised with injected clocks and the lifecycle integration test; watch a real awaiting circling or hard-cap stop and confirm the notice arrives after the arbiter's hand-on, not at the three-minute report wait.
- After the verify fix to the driver stencil (it names no recipe row), `lyx gate test --tags integration ./cmd/lyx` passed; the full `lyx gate test ./...` and `--tags integration ./...` runs are the pre-push check.
