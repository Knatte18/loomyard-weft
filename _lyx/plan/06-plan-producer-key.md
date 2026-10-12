# Card 6 — plan-producer-key

**Create:**
- `internal/loomengine#PlanProducerSingle`
- `internal/loomengine#PlanProducerTwill`

**Edit:**
- `internal/loomengine#Config`
- `internal/loomengine#LoadConfig`
- `internal/loomengine/template.yaml`
- `internal/loomengine#templateConfig`
- `internal/loomengine#TestLoadConfig_Loads`
- `internal/loomengine#TestLoadConfig_Refuses`
- `contracts/specs/refusal-spec.md`
- `docs/overview.md`

**Intent:**
D1: `loom.yaml` gains `plan_producer`, accepting `single` and `twill`, read by Plan-Write only (card 9).

- `Config` gains `PlanProducer string` with tag `yaml:"plan_producer"`, beside `DiscussionProducer`.
- `PlanProducerSingle` and `PlanProducerTwill` are declared in a `const` block of their own beside the `DiscussionProducer*` block, each with a doc line.
- `LoadConfig` turns an empty `PlanProducer` into `PlanProducerSingle` (an explicit `plan_producer: ""` reads as `single`, unlike `discussion_producer`, which refuses it) and refuses any other value than the two with `fmt.Errorf("loom config key %q: unknown value %q; set it to %q or %q", "plan_producer", …)`, after the `discussion_producer` switch.
  An absent key needs no code: `configengine`'s in-memory fill of a present file's missing template keys gives it the template's `single`.
- `internal/loomengine/template.yaml` gains, after `plan_timeout_min`, `plan_producer: single` with a comment: the producer the Plan-Write row runs; `single`, the default, runs the single-agent writer; `twill` runs the single writer, which forks itself once per lens of the `twill-plan` fan in `burler.yaml` to review its plan before it finishes; an empty value reads as `single`.
  With the template's `single`, a hub that does not opt in renders today's plan prompt (card 9 pins the bytes).
- The file header of `config.go`, the file holding `Config` and `LoadConfig`, gains one line: `plan_producer` selects the Plan-Write row's producer, `single` or `twill`, and an empty value reads as `single`.
- `contracts/specs/refusal-spec.md` gains the loom row `| config: unknown plan_producer | loom.yaml's plan_producer is neither single nor twill | correctness halt | set plan_producer to single or twill in loom.yaml |`, formatted like the `discussion_producer` row after it (`PATTERN-refusal-way-forward`).
- `docs/overview.md`'s loom config line (the `loom.yaml` key list) names `plan_producer`, which is `single` or `twill` and selects the Plan-Write row's producer, beside `discussion_producer`.

Tests:
- `templateConfig`, the test's expected template `Config`, gains `PlanProducer: "single"`, so the "template defaults" row and the legacy-contents row (a file seeded without the key) pin that an absent key loads as `single`.
- `TestLoadConfig_Loads` gains the rows "plan_producer twill" (`values: {"plan_producer": "twill"}`, `c.PlanProducer = "twill"`) and "empty plan_producer reads as single" (`values: {"plan_producer": "\"\""}`, `c.PlanProducer = "single"`).
- `TestLoadConfig_Refuses` gains `{"unknown plan_producer", "plan_producer", "sideways", []string{`"single"`, `"twill"`, "set it to"}, ""}`, which reaches the refusal past every upstream check.

Acceptance: "`plan_producer: twill` loads; an empty or absent key loads as `single`; an unknown value fails with an error naming `single` and `twill`."

**ImpactSummary:** one new config key with a refusing validation; every existing `loom.yaml` loads unchanged as `single`.

**Commit:** `6: Add loom.yaml plan_producer, single by default`
