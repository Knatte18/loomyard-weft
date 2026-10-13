# Card 6 — loom-hard-cap-config

**Edit:**
- `internal/loomengine#Config`
- `internal/loomengine#ConfigSpec`
- `internal/loomengine#LoadConfig`
- `internal/loomengine/template.yaml`
- `internal/loomcli#loomCLI.wire`
- `contracts/specs/refusal-spec.md`
- `docs/overview.md`
- `internal/loomengine#templateConfig`
- `internal/loomengine#TestLoadConfig_Loads`
- `internal/loomengine#TestLoadConfig_Refuses`
- `internal/loomcli#TestWire_ReviewKeysReachTheEnv`
- `internal/configreg#TestPrepareRunSettings`

**Create:**
- `plan:internal/loomengine#Perch` -> `type Perch string`
- `plan:internal/loomengine#PerchDiscussion` -> `const PerchDiscussion Perch = "discussion"`
- `plan:internal/loomengine#PerchPlan` -> `const PerchPlan Perch = "plan"`
- `plan:internal/loomengine#PerchWebster` -> `const PerchWebster Perch = "webster"`
- `plan:internal/loomengine#PerchMerge` -> `const PerchMerge Perch = "merge"`
- `plan:internal/loomengine#PerchCaps` -> `type PerchCaps struct`
- `plan:internal/loomengine#Config.Caps` -> `func (cfg Config) Caps(p Perch) PerchCaps`
- `plan:internal/loomengine#CapRoute` -> `func CapRoute(dirs configengine.Dirs, key string) (string, error)`
- `plan:internal/loomcli#loomPerches` -> `var loomPerches map[string]loomengine.Perch`
- `plan:internal/loomcli#perchHardMaxBounces` -> `func perchHardMaxBounces(cfg loomengine.Config) map[string]int`

**Intent:** loom config gains a hard cap per perch that no round passes (D3), refused at load when it cannot hold, and wired into each loom Bouncer row.

`Config` gains, beside the budget keys, `ReviewHardMaxBounces` (`review_hard_max_bounces`) and the per-perch `DiscussionHardMaxBounces`, `PlanHardMaxBounces`, `WebsterHardMaxBounces` and `MergeHardMaxBounces` (`<perch>_hard_max_bounces`).
A doc comment says a per-perch 0 takes `review_hard_max_bounces`.
`template.yaml` adds `review_hard_max_bounces: 4` after `review_max_bounces` and the four per-perch keys at 0 after their budget keys, each commented.
The comments say a round numbered at or past the hard cap is never continued, that it is accepted or escalated `hard-cap`, and that 0 takes the run-wide value.
The existing budget comments change "a spent budget escalates to the parent" to "a spent budget runs the soft-cap rule (the judge's fix tags settle the round, escalating only a stuck run)".
`review_circling_checkpoint`'s and the three per-perch budget comments' "reaches the budget escalation" becomes "reaches the soft-cap rule".
The same phrase in `LoadConfig`'s checkpoint-above-budget code comment and in `TestLoadConfig_Loads`' doc changes the same way.
The auto-accept comments say the perch accepts a harvested round at a spent budget instead of running the rule, and `merge_auto_accept`'s says an open BLOCKING finding sends the round to the rule.
`ConfigSpec` classifies all five new keys `none`, as the `*_max_bounces` keys are, so `run_settings` can set them.

`Perch` names a loom review perch by its key prefix, with constants `PerchDiscussion`, `PerchPlan`, `PerchWebster` and `PerchMerge`.
`PerchCaps` holds `SoftKey string`, `Soft int`, `HardKey string` and `Hard int`: a perch's resolved soft and hard cap and the loom.yaml key each resolves from.
`Config.Caps(p)` resolves both: the perch's own `<perch>_max_bounces` when above 0, else `review_max_bounces`, and the same for the hard keys.

`CapRoute(dirs, key)` returns the way forward that changes `key`, by the layer its resolved value comes from, read through `configengine.StackLayers(dirs, ConfigSpec())` `Origins`, as `internal/batcher` reads an origin.
The module name is `loom`.

- Local layer: `run "lyx config loom --layer local --set <key>=<n>" in the task worktree, and correct the task's run_settings with "lyx board upsert" too, so a later Seed-Child writes the same value`.
- Worktree layer: `run "lyx config loom --set <key>=<n>" in the task worktree`.
- Default layer, or no layer, meaning the template: `run "lyx config loom --layer default --set <key>=<n>" from the hub's prime worktree, where the operator or the hub orch runs it` (the record's added decision on the hub-default route).

In each text the key is spelled out and `<n>` stays literal.
A `StackLayers` error is returned.

`LoadConfig`:

- After the at-least-1 loop beside `review_max_bounces`, a `review_hard_max_bounces` below 1 is refused through `CapRoute`, never with that loop's "set it to a positive integer in loom.yaml" text: "loom config key %q: must be at least 1, got %d; way forward: <CapRoute(dirs, \"review_hard_max_bounces\")>, then run \"lyx loom resume\"".
- After the per-perch budgets' negative loop, each of the four per-perch hard keys below 0 is refused through `CapRoute`, never with that loop's "..., in loom.yaml" text: "loom config key %q: must not be negative, got %d; set it to 0 to take review_hard_max_bounces, or to a positive integer; way forward: <CapRoute(dirs, that key)>, then run \"lyx loom resume\"".
  A `run_settings` value lands in the local layer, so only the route of the layer it comes from clears it.
- After both, for each of the four perches, a resolved `Hard` below its resolved `Soft` is refused.
  The message names both keys and both values: "loom config key %q: resolved hard cap %d is below %q's resolved soft cap %d; way forward: raise the hard cap: <CapRoute(hard key)>, or lower the soft cap: <CapRoute(soft key)>, then run \"lyx loom resume\"".
  Equal caps load (D3).
- The file header comment lists the new keys, their refusals and the hard-below-soft refusal.
  Its Merge-Review clause "which the Bouncer escalates instead of accepting while its ledger holds an open BLOCKING finding" becomes "which sends a round whose ledger holds an open BLOCKING finding to the soft-cap rule instead of accepting it" (D8).

`loomPerches` maps the four loom Bouncer row names (`loomshed.NameDiscussionBouncer` and so on) to their `Perch`.
`perchHardMaxBounces(cfg)` maps each of those rows to `cfg.Caps(perch).Hard`, the resolved value, unlike `perchMaxBounces`, whose 0 is resolved downstream.
`loomCLI.wire` fills `RowHardMaxBounces: perchHardMaxBounces(loomCfg)` beside `RowAutoAccept`.

`refusal-spec.md`:

- The "config: review key below 1" row is unchanged: the hard keys take the new row's per-layer way forward, not its static one.
- A new row "config: hard cap out of range" beside it: at load, `review_hard_max_bounces` is 0 or negative, a per-perch hard cap is negative, or a perch's resolved hard cap is below its resolved soft cap, a correctness halt.
  Its way forward: set the hard cap to a positive integer (a per-perch hard cap to 0 to take `review_hard_max_bounces`), or for hard below soft raise the hard cap or lower the soft cap, in the layer the key comes from, the message naming the command (`lyx config loom --layer local --set` in the task worktree plus `lyx board upsert` for `run_settings`, `lyx config loom --set` in the task worktree, or `lyx config loom --layer default --set` from the prime), then `lyx loom resume`.

`docs/overview.md`'s loom config sentence lists `review_hard_max_bounces` and the per-perch hard keys, where 0 takes the run-wide value, and the hard-below-soft refusal.

Tests:

- `templateConfig` gains `ReviewHardMaxBounces: 4` and the four per-perch hard fields at 0.
- `TestLoadConfig_Loads`' "explicit review checkpoint and budget" row adds `review_hard_max_bounces: 7`, since its budget of 7 would now be refused.
  A new row "explicit hard caps" sets `review_hard_max_bounces` 5 and `plan_hard_max_bounces` 6 and expects both fields.
  A new row "a hard cap equal to the soft cap loads" sets `review_max_bounces` and `review_hard_max_bounces` both to 3 (D3).
- `TestLoadConfig_Refuses` gains rows for:
  - `review_hard_max_bounces` 0 in a worktree-only config, wanting "must be at least 1", `lyx config loom --set review_hard_max_bounces=<n>` and `lyx loom resume`;
  - each per-perch hard key at -1 in a worktree-only config, wanting "set it to 0 to take review_hard_max_bounces", `lyx config loom --set <that key>=<n>` and `lyx loom resume`;
  - `review_max_bounces` 5 under the default hard cap of 4 in a worktree-only config, wanting `review_hard_max_bounces`, `review_max_bounces`, `lyx config loom --set review_hard_max_bounces=<n>`, "in the task worktree" and `lyx loom resume`;
  - `webster_max_bounces` 5, wanting `review_hard_max_bounces` and `webster_max_bounces`.
  A new subtest "a hard cap below the soft cap names the route of the layer each key comes from" runs over a hub stack with a board default from `loomTemplateWith`:
  - a local `review_max_bounces: 5` wants `lyx config loom --layer local --set review_max_bounces=<n>`, `lyx board upsert` and, for the hard key from the board default, `lyx config loom --layer default --set review_hard_max_bounces=<n>` and "prime";
  - a worktree `review_hard_max_bounces: 2` under the board default's soft cap of 3 wants `lyx config loom --set review_hard_max_bounces=<n>` and "in the task worktree";
  - a local `review_hard_max_bounces: 0` wants "must be at least 1", `lyx config loom --layer local --set review_hard_max_bounces=<n>`, `lyx board upsert` and `lyx loom resume`.
  A local `run_settings` value reaches each refusal with the local route this way.
- `TestWire_ReviewKeysReachTheEnv` seeds `plan_hard_max_bounces: 5` and asserts `c.env.RowHardMaxBounces` equals 4 for the Discussion, Webster and Merge Bouncer rows and 5 for Plan-Bouncer.
- `TestPrepareRunSettings` gains a row for the small-task advice's settings `{"loom": {"review_max_bounces": 2, "review_hard_max_bounces": 3}}`.
  It expects a loom document holding both keys in the order the existing renderer emits, which proves the hard-cap key settable through `run_settings`.

**ImpactSummary:** Five new loom.yaml keys with load-time refusals; every config whose resolved budget is above 4 with no hard-cap key now refuses at load (the record's open risk); the loom wiring adds one Env map, and no existing caller of `LoadConfig` changes.

**Commit:** 6: loom.yaml hard caps per perch, refused below 1 or below the soft cap
