# Card 3 — design-prefix-refusal

**Create:**
- `plan:internal/boardengine#checkDesignPrefix` -> `func checkDesignPrefix(prefix string) error`
- `plan:internal/boardengine_test#TestLoadConfig_DesignPrefixRefused` -> `func TestLoadConfig_DesignPrefixRefused(t *testing.T)`

**Edit:**
- `internal/boardengine#LoadConfig`
- `internal/boardengine/doc.go`

**Uses:**
- `plan:internal/boardengine#decisionFilePrefix`
- `internal/boardengine_test#writeBoardConfig`

**Intent:**
Keep the render from ever owning a record file (D1).

`LoadConfig` calls `checkDesignPrefix(cfg.DesignPrefix)` after the label checks.
It refuses a prefix that is a prefix of `decisionFilePrefix`, the empty prefix included, and a prefix that starts with `decisionFilePrefix`, since either renders a design file onto a `decision-<slug>.md` name.
The refusal reads: `board.yaml: design_prefix %q would name a design file decision-<slug>.md, the name of an entry's decision record: set design_prefix in board.yaml to another value, such as "design-", with "lym config board --set design_prefix=design-"`.
`lym config board` edits the file without `LoadConfig`, so the named verb still runs while the board refuses.
The template's default `design-` and every fixture prefix in the tree (`proposal-`, `d-`, `design-`) pass.

`doc.go`'s `# Decision record` section adds the refusal in one sentence.

`TestLoadConfig_DesignPrefixRefused` (tier 0, external package) is a table over `writeBoardConfig` fixtures.
- `decision-`, an empty `design_prefix: ""`, `dec` and `decision-x-` are each refused, and the error contains `design_prefix` and `set design_prefix in board.yaml`.
- `design-` loads.

**ImpactSummary:** One extra check in LoadConfig, run by every board config load in the hub. It refuses only prefixes that would collide with a record file name, and no prefix in the template or any test fixture collides.
