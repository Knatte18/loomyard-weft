# Card 7 — twill-config-value

**Create:**
- `plan:internal/loomengine#DiscussionProducerTwill` -> `const DiscussionProducerTwill = "twill"`

**Edit:**
- `internal/loomengine#LoadConfig`
- `internal/loomengine/template.yaml`
- `internal/loomengine#TestLoadConfig_Loads`
- `internal/loomengine#TestLoadConfig_Refuses`

**Uses:**
- `internal/loomengine#DiscussionProducerSeats`

**Intent:**
Accept `discussion_producer: twill` (decision record D1).

`DiscussionProducerTwill` joins the constant block beside `DiscussionProducerSeats`, documented as selecting the single writer with lens forks.
`LoadConfig`'s producer switch admits it, and the unknown-value error names all three values as the way forward (`PATTERN-refusal-way-forward`).
The file header's sentence about `discussion_producer` names the three values.

`template.yaml`'s `discussion_producer` comment gains the third value: `twill` runs the single-agent writer, which forks itself once per lens of the `twill-discussion` fan in `burler.yaml` to review its record before it finishes.

`TestLoadConfig_Loads` gains a row loading `discussion_producer: twill`;
`TestLoadConfig_Refuses`' unknown-producer row wants `"twill"` named beside `"single"` and `"seats"`.

The producer behaves as `single` until card 8; `docs/overview.md` describes the value there, where the behavior lands.

**ImpactSummary:** The switch in `LoadConfig` and one error string; `loomrecipe` and `loomcli` compare against `DiscussionProducerSeats` only and keep treating `twill` as the single writer's engine.
