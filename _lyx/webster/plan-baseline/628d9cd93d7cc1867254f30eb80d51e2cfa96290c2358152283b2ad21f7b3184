# Card 1 — loom-alignment-config

**Edit:**
- `internal/loomengine#Config`
- `internal/loomengine#ConfigSpec`
- `internal/loomengine#LoadConfig`
- `internal/loomengine/template.yaml`
- `internal/loomengine#templateConfig`
- `internal/loomengine#TestLoadConfig_Loads`
- `internal/loomengine#TestLoadConfig_Refuses`
- `internal/configreg#TestModules_EveryTemplateKeyClassified`
- `internal/configreg#TestPrepareRunSettings`
- `contracts/specs/refusal-spec.md`
- `cmd/lyx/testdata/test-budget.yaml`

**Create:**
- `internal/loomengine#AlignmentSettings`
- `internal/loomengine#ResolveAlignment`
- `internal/loomengine#TestResolveAlignment`

**Intent:**
Add D7's four `loom.yaml` keys, classified for the config layers, and the one cross-key check.

`Config` gains `Alignment string` (`alignment`), `AlignmentTimeoutMin int` (`alignment_timeout_min`), `SkipAlignment bool` (`skip_alignment`) and `AlignmentPRApprove bool` (`alignment_pr_approve`), grouped under one comment beside the arbiter's keys.
`template.yaml` gains the four lines with their defaults and comments: `alignment: opus[high]` (empty takes `opus[high]`), `alignment_timeout_min: 20` (0 takes 20, kept below `parent_review_wait_min`), `skip_alignment: false` (true sends parent review and PR-Gate to the run's parent exactly as before the agent) and `alignment_pr_approve: true` (false makes an agent approve at PR-Gate an escalation, so the parent gives the yes).
`ConfigSpec` classifies `alignment`, `skip_alignment` and `alignment_pr_approve` as `CategoryGuard`, and `alignment_timeout_min` as `CategoryNone`.
The reason goes on the `ConfigSpec` doc comment: the agent's approve at PR-Gate lands on the parent with no later gate, so a weaker model, a re-opened auto-landing or a local `skip_alignment: false` under a hub `true` hollows the approval gate, while a shorter timeout only routes a review to the parent.

`LoadConfig` validates `alignment` as a model-spec only when non-empty, like `arbiter`, and refuses a negative `alignment_timeout_min` with the way forward "set it to 0 for the 20 minute default, or to a positive integer below parent_review_wait_min".
After the `parent_review_wait_min` check it refuses a resolved timeout (0 resolves to 20) at or above `parent_review_wait_min`.
The refusal names both keys, both values and the way forward: set `alignment_timeout_min` below `parent_review_wait_min`, or raise `parent_review_wait_min` in the hub's `loom.yaml`.
The check runs on the merged layers, so a local layer, which is also where a task's `run_settings` land through Seed-Child, is checked like the hub file.
The file header comment gains a sentence naming the four keys.

`ResolveAlignment` is `ResolveArbiter`'s twin in `driver.go`: it parses and resolves the model-spec (empty takes `opus[high]`) and returns `AlignmentSettings{Model, Effort, Version, Timeout}`, with a zero `AlignmentTimeoutMin` taking 20 minutes.
A spec that does not parse or resolve is an error naming the `alignment` key.

Tests:
- `templateConfig` carries the four template values.
- `TestLoadConfig_Loads`: its `explicit parent_review_wait_min` row, which sets the wait to 5, also sets `alignment_timeout_min` to 4, since the default 20 is now refused at a 5-minute wait. A new row sets all four keys and asserts them on the decoded `Config`. A new subtest loads a local `alignment_timeout_min: 10` over the hub template and gets 10, since the key is open to the local layer.
- `TestLoadConfig_Refuses`' row struct gains an `extra map[string]string` field. A row that sets it is written through `writeLoomConfigWithKeys` with its key added, and every existing row leaves it nil. New rows cover a malformed `alignment` spec, a negative `alignment_timeout_min`, and the coherence check twice: an explicit `alignment_timeout_min` equal to `parent_review_wait_min`, and `alignment_timeout_min: 0` (resolving to 20) with `extra` `{parent_review_wait_min: 20}`. Each wants both key names in the error.
- `TestLoadConfig_Refuses` also gets two local-layer subtests, written like its existing `a local parent_review_wait_min is refused` subtest. A local `skip_alignment` is refused naming the local file and the key; it stands for the three guard keys, whose deny list `TestModules_EveryTemplateKeyClassified` pins. A local `alignment_timeout_min: 60` over the hub's 60-minute wait is refused by the coherence check, which is the `run_settings` route, since Seed-Child writes `run_settings` as the local layer.
- `TestModules_EveryTemplateKeyClassified`'s `wantLocalDenied["loom"]` adds `alignment`, `alignment_pr_approve` and `skip_alignment` in sorted order.
- `TestPrepareRunSettings` gains a row refusing `run_settings` that set `skip_alignment`, and a row accepting `alignment_timeout_min`.
- `TestResolveAlignment` covers the empty default, an explicit spec, the 20-minute default and an explicit timeout, and an unresolvable spec naming `alignment`.

`refusal-spec.md`'s loom section gains three rows: `config: alignment not a model-spec`, `config: alignment_timeout_min negative` and `config: alignment_timeout_min at or above parent_review_wait_min`, each with the way forward the message names.
`test-budget.yaml` raises `internal/loomengine` by one for `TestResolveAlignment`.

**ImpactSummary:** Four new loom keys and a load-time check: every caller of `LoadConfig` sees the new refusal, and only a hub or worktree whose `parent_review_wait_min` is 20 or less is affected; no other package changes.
