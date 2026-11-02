```yaml
lens: forced-edits
outcome: reported
```

Task: a forced-edits lens review of the cold-gate-fixer plan, covering the overview and cards 1 to 13.

**Findings**

1. **BLOCKING — Card 4 (gate-fixer-config), D3: `fixer_timeout_min` is never classified.** Card 4 adds the key to `internal/gateslot/template.yaml` but does not list `internal/gateslot#ConfigSpec`, whose `Categories` map must cover every template unit.
   - Evidence: `internal/gateslot/config.go:104-130` (ConfigSpec `Categories` has `slots`, `tier_parallel`, `busy_quit_sec`, `tiers`, `gates`, `scoper`, `refuse_raw`, `build`, `inert_paths`; no `fixer_timeout_min`). `internal/configengine/doc.go:72`: "a hub-wide module also refusing a path no entry covers"; gate is `HubWide: true`. `internal/configreg/configreg_test.go:104`: "template unit %q of %s is not covered by a Categories entry; classify it".
   - Fix: add `internal/gateslot#ConfigSpec` under `Edit` with a `fixer_timeout_min` entry. `fixer` inside `gates` is already covered by the `gates` guard entry.

2. **BLOCKING — Card 5 (module-fixer-keys), D3: the three new keys are never classified.** Each module's spec keeps a `Categories` map that must cover every template key, and card 5 lists none of the spec builders. Without them `configreg_test` fails.
   - Evidence: `internal/darnengine/config.go:62-66` (`writer`, `writer_timeout_min`, `verify_attempts`, `tree_scope` only). `internal/loomengine/tapestry.go:143-158` (explicit `plan`, `plan_timeout_min`, … list). `internal/loomengine/config.go:544` (explicit `discussion`, … list). `internal/configreg/configreg_test.go:104`.
   - Fix: add `internal/darnengine#ConfigSpec`, the tapestry spec function at `internal/loomengine/tapestry.go:~140` and `internal/loomengine#ConfigSpec` to card 5's `Edit` list, with `gate_fixer`, `plan_gate_fixer` and `discussion_gate_fixer` entries (CategoryNone, like their sibling model-spec keys).

3. **MEDIUM — Card 11/12, D4: the `gate-fixer` strand's own gate call goes through an unchecked keep-warm path.** A fixer strand's own `lym gate test` call is capped by `shuttleengines.KeepWarm`, which resolves `engine.KeepWarmCaps(baseRole(role))` for the new role `gate-fixer`. No card checks or registers how an engine's keep-warm caps answer an unknown role. If it errors, the call falls back to `DefaultAwaitCap` with a Warn on every fixer gate call.
   - Evidence: `internal/shuttleengines/keepwarm.go:58-62`. `internal/gatecli/detachfront.go:253-256` (the fallback is logged at Warn).
   - Fix: find the engine's per-role keep-warm table and add `gate-fixer` on card 1 or card 11.

4. **LOW — D8, tokencount: no card covers counting the `gate-fixer` role apart.** D8 says the `gate-fixer` role lets `tools/tokencount` count fixer runs apart. tokencount maps roles and kinds to segments (`tools/tokencount/segment_test.go:32` row `{"recovery", "Webster", "recovery"}`, `tools/tokencount/profile.go:250` keyed on kind `recovery`), and no card adds a `gate-fixer` mapping. If the table is closed, fixer runs land unclassified.
   - Evidence: the files above; no card names `tools/tokencount`.

5. **LOW — Card 3, D6: webster comments and user-facing text still name "verify-fixer session".** These spots name the role as a literal string, not through the constant card 3 moves into `gatefixer` (`gatefixer.RoleVerifyFixer`). The text stays true, but `PATTERN-named-constants` would want the shared constant where the role is load-bearing.
   - Evidence: `internal/websterengine/contractevidence.go:46` (`writerRecovery = "a recovery or verify-fixer session"`) and `internal/websterengine/recoverbatch.go:548`. Card 3 lists neither file.
   - No change is forced, because the role string is unchanged.

**Tool calls beyond the inherited context**
1. Grepped the shedrecipe import allowlist, the callers of `PlanTable`/`DiscussionTable`/`planLensDirective`, the `RenderForkPrompt`/`RenderRecoveryPrompt` callers, the `runnerCmd`/`runDetached` callers, the `verify-fixer` literals and the tokencount role mentions, to check caller coverage; the earlier reads did not include these call sites.
2. Read gateslot `ConfigSpec`, the stencil registry's family handling, the darn/loom/tapestry test function names and the `NewRunner` call sites, to check config registration and test glyphs; not read before.
3. Grepped configengine/configreg for `Categories` enforcement and the darn/tapestry/loom spec maps, to confirm the key-category rule.
4. Read the darn and tapestry `ConfigSpec` bodies, to confirm their category maps are explicit lists.
5. Grepped `segmentcolor` segments, the shuttleengines keep-warm functions and the tools dir, to check the segments card 8 names and the keep-warm role path.
6. Read `shuttleengines/keepwarm.go` and the tokencount role mappings, to judge how a new role is resolved.
