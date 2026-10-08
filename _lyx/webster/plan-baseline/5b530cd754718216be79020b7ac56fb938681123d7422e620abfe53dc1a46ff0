# Card 10 — template-review-models

**Edit:**
- `internal/loomengine/template.yaml`
- `internal/loomengine/config_test.go`
- `internal/loomcli/wiring_test.go`

**Intent:**
Apply the parent decision "Template review models per segment" (decision record, "Added after Discussion (parent, 2026-10-08)"), which no card of the retired generation picked up (PR finding F1).

- `template.yaml`: `review: opus[medium]`, since Discussion-Review and Plan-Review each run one solo reviewer, and `webster_review: sonnet[high]`, since code review runs on sonnet.
  `discussion_review`, `discussion_fix`, `plan_review`, `plan_fix` and `webster_fix` stay `""` and inherit the run-wide `review` or `fix` list; `fix` stays `opus[medium]` and `judge` stays `sonnet[medium]`.
  Each changed line keeps its existing key comment, which already states what the key does and how an empty value falls back.
- `config_test.go`: `templateConfig` pins `Review` at `ModelSpecList{"opus[medium]"}` and `WebsterReview` at `ModelSpecList{"sonnet[high]"}`, the other five per-segment lists staying `ModelSpecList{""}`.
  The `TestLoadConfig_Loads` row "keys absent from a seeded file take template defaults" needs no change: its legacy file sets `review` itself, and the absent `webster_review` takes the new template value through `templateConfig`.
- `wiring_test.go`: `TestWire_ReviewKeysReachTheEnv` seeds the template with `plan_review` set, so the Webster-Burler row no longer equals the run-wide models.
  Its loop over rows keeping the run-wide models covers `loomshed.NameDiscussionBurler` alone, and a new assertion pins that the Webster-Burler row's `Review` is one sonnet entry of effort high, from the template's `webster_review`, while its `Fix` is the run-wide `c.env.ReviewModels.Fix`.
  The test's doc comment states both.

Grep the tree for any other test pinning the template's `review: sonnet[medium]` or an empty `webster_review`, and move it with this card.

**ImpactSummary:** Template defaults and the two tests that pin them; a seeded loom.yaml that already sets review or webster_review keeps its value.

**Commit:** 10: loomengine: template review on opus, webster_review on sonnet[high]
