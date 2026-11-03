# Card 3 — shedbuild-block-row-parse

**Edit:**
- `internal/shedbuild#Row`
- `internal/shedbuild#Recipe`
- `internal/shedbuild#checkRowShape`
- `internal/shedbuild/doc.go`
- `contracts/specs/shed-recipe-spec.md`
- `internal/shedbuild#TestParse_ValueAndErrorCases`

**Uses:**
- `plan:internal/shedrecipe#EngineBlock`

**Intent:** Add the block row and `profile_recipe` to the recipe format, parsed only by `Parse` (D1, D8, PATTERN-sole-parsers).

`Row` gains:
- `Block string` (`yaml:"block"`), the name of the recipe the block runs;
- `BlockGate string` (`yaml:"block_gate"`), the name of its gate check;
- `SkipTo string` and `BlockPath string`, both `yaml:"-"`: outside the YAML format, set only by `Expand` (card 4), so `KnownFields(true)` refuses either written in a file.

`Recipe` gains `ProfileRecipe string` (`yaml:"profile_recipe"`): the recipe whose run-profile block a task takes, empty meaning the recipe's own name.

`checkRowShape` treats a row with `block` or `block_gate` set as a block row, and refuses, each naming the row's index and name and a way forward:
- a block row that also sets `engine`, `config`, `on_stuck`, `on_reroute`, `segment`, `max_bounces` or `entry_gate`: `shedbuild: producer %d %q: a block row carries only name, block, block_gate and on_done, and sets %s; way forward: remove %s, since the block's rows carry their own`;
- a block row missing `block`, and one missing `block_gate`: `...: a block row needs %s; way forward: set %s, block_gate naming one of the shipped checks`;
- a row whose `engine` is `shedrecipe.EngineBlock`: `...: engine Block is reserved for block rows; way forward: write the row as a block row with block and block_gate`.
The empty-engine refusal applies to every row that is not a block row.
A block row's `on_done` may be empty here; `Expand` judges it (card 4).

`doc.go` describes the block row's keys and refusals and the `profile_recipe` key.
`shed-recipe-spec.md`'s Document shape paragraph gains the top-level `profile_recipe` key and the block row, which carries only `name`, `block`, `block_gate` and `on_done` and no engine, and its row-key sentence is rewritten to name engine rows and block rows.

Tests, tier0: `TestParse_ValueAndErrorCases` gains rows for a well-formed block row decoding into `Block` and `BlockGate` with an empty `on_done`, for `profile_recipe` decoding into `ProfileRecipe`, for each refused key on a block row (one row per key), for a missing `block` and a missing `block_gate`, for a hand-written `engine: Block`, and for `block_path` written in a file refused as an unknown field.

**ImpactSummary:** Additive format keys and three new refusals inside the sole parser; every shipped recipe sets none of the keys, so each parses as before.
