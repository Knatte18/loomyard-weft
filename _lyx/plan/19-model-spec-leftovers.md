# Card 19 — model-spec-leftovers

**Edit:**
- `contracts/specs/llm-model-spec.md`
- `internal/modelspec/template.yaml`
- `internal/landingshed/template.yaml`
- `internal/loomengine/template.yaml`
- `internal/landingshed/config_test.go`

**Intent:** Close the model-spec leftovers (D11).

`llm-model-spec.md` states the short form `model[effort]` as canonical, and every example uses it.
For example `sonnet[high]`, `opus[max]`, `claude:claude-sonnet-4-5[high]`, and the precedence example's `sonnet[high]`.
Explain why the "effort comes from the registry default" outcome holds in short-form words.
A sentence says the long form `key=value` for `effort` stays parseable.
A non-effort parameter such as `v=4.5` keeps its `key=value` spelling, since the short form names effort alone.
The parser is not changed.

Every template comment's `docs/reference/model-spec.md` link points at `contracts/specs/llm-model-spec.md`.
Grep the tree for `docs/reference/model-spec.md` afterwards; any remaining hit joins this card.
If a test pins a template's bytes (a template-drift or embed test), update its expectation in this card.

`internal/landingshed/config_test.go` drops every long-form spec.
Its well-formed fixtures use the short form.
Its malformed-input fixtures become malformed short-form specs that hit the same failure paths, for example an unclosed `"opus[high"` and `"sonnet[medium"`, each still asserting the same named field in the error.

**ImpactSummary:** Doc, YAML-comment and test-fixture rewording; parsing behavior and template values are unchanged.
