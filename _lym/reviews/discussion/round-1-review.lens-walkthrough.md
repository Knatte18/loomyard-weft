```yaml
lens: walkthrough
outcome: reported
```

Lens: walkthrough. I ran each decision's mechanism end to end against the code in context.

**Decision walkthroughs**

- **D1.** Case: a `hardener` recipe with `verify_source: tree_scope`, armed by loomcli. `verifySource` sends every tree_scope recipe to `darnVerifySource` (`internal/loomcli/arm.go:351`). D1 states the branch it adds, so the purpose is met.
- **D2.** Case: a task with `run_settings {"hardener":{"packages":[]}}`. `PrepareRunSettings` takes the module once D9 registers it, and Preflight refuses the empty value. Purpose met.
- **D3.** Case: three models, cap 7. The cyclic expansion feeds `pickRound`, which clamps (`internal/burlerengine/models.go:25-31`). Rounds come out 1,2,3,1,2,3,1. Purpose met.
- **D4.** Case: `llm_driving` false, `concurrency` 3. A serial tier 2 run, then `--copies 3` inside the held slots. Purpose met. `tmuxkit.Main` keeps the tests' reed servers in a private `TMUX_TMPDIR`, so the sandbox reap never sees them.
- **D5.** Case: `sandbox new` commits `loom.yaml {selfreport: false, friction: ""}`. Only `friction: ""` turns anything off; `selfreport` is a retired key that is ignored. **Not fully met** (finding 2).
- **D5 and D6, the stray report.** Case: one round with a leftover hub at the end of the review row. Four reaps per round write "the round's stray report". **Not met** (finding 4).
- **D6, convergence.** Case 1: a `reviewer` list of one entry, set through `run_settings`. Case 2: a round whose review is clean. **Not met** (findings 3 and 5).
- **D7.** Case: the end of the fix row. The order of the gates, the end-of-row reap and the archive is not stated (finding 4).
- **D8.** Case: a hub `agents.yaml` whose `skills:` names `treadle-judge`, loaded after the role is deleted. `Load` refuses it. **Not met** (finding 1).
- **D9.** Case: the defaults load, and `run_settings` overrides `reviewer`. This works, but it feeds case 1 of D6.
- **D10.** Case: on_done path Preflight → Record-Write → Harden-Bouncer → Describe. The fix row is placed at Harden-Bouncer, after Record-Write, and a second `HardenFix` row is refused by `onlyRowOf`. Purpose met.
- **D11 and D12.** Case: `lym harden sandbox new` from the root, and the doc sweep. Purpose met, apart from what the other lenses own.

**Findings**

1. **BLOCKING · design · D8, and the acceptance criterion "a hub `agents.yaml` naming `treadle-judge` still loads"**
   - D8 says `internal/agentconfig` refuses no unknown role. The code does refuse one.
   - Evidence:
     - The `Load` doc in `internal/agentconfig/config.go` says "It refuses a role key outside the roles lym starts".
     - `skillLists` returns `unusable(skillsKey, "names %q, which is no role lym starts")`.
     - `internal/agentconfig/config_test.go:89`: the row `{"UnknownRole", "skills:\n  nobody: []\n", ""}` expects `ErrUnusable`.
   - Deleting the `treadle-judge` and `treadle-targeting` roles therefore breaks any hub whose `agents.yaml` names them. That needs the `configreg.Reinterpretation` that PATTERN-config-layers requires, which D8 drops.
   - The support log's bound-breaker "fix" (`support-log.md:45`) rests on the same false claim.

2. **MEDIUM · design · D5, the `loom.yaml` override, and the criterion on `selfreport: false`**
   - `selfreport` is a retired `loom.yaml` key that is ignored. Evidence: `internal/loomengine/config_test.go:212`, case "retired selfreport key ignored".
   - The automatic upstream path is the Tier 2 friction reflection. `friction` empty turns it off (`internal/loomengine/template.yaml:37`). That reflection files only through `lym selfreport create` (`internal/friction/doc.go:49`).
   - So half of the committed override does nothing, and the criterion pins a dead key.
   - The rationale should rest on `friction: ""` alone.

3. **MEDIUM · design · D6's convergence rule, together with D9 and D2's `run_settings` override of `reviewer`**
   - CONVERGED requires "a reviewer model different from the previous round's".
   - A `reviewer` list with one entry, or with repeated adjacent entries after the cyclic expansion of D3, can never satisfy that. Such a list is legal through `hardener.yaml` or `run_settings`.
   - Such a campaign can only end capped. No bound, refusal or Preflight check is stated.

4. **MEDIUM · consistency · D5, D6 and D7: the stray report and the archive order**
   - Each round has four reaps: the start and end of the review row, and the start and end of the fix row.
   - D6 names one per-round stray report and gives no rule for append or overwrite. An overwrite loses the review row's leftovers before the judge reads the facts.
   - D7 archives "after its gates" at the end of the fix row, but does not order the archive against D5's end-of-row reap. If the archive copies first, the fix row's own end reap never reaches `_lym/harden/round-<n>/`.

5. **LOW · design · D6's convergence needs "a green live gate" in the clean round**
   - The record does not say that `HardenFix` runs its `live` gate when the round's review holds no findings.
   - If the fix half skips the fixer and its gates on a clean review, that round has no live report. The facts can then never show the clean review and the green gate together.
   - I did not find burler's clean-review fix-half behaviour in the code I read (`internal/burlerengine/doc.go` grep). This is a fact the design rests on that the record does not settle.

**Tool calls**

1. `contracts/recipes/darn-recipe.yaml` row edges: to walk the ring order (Bouncer → review → fix → Bouncer); the manifest lacked the recipe's edges.
2. `internal/burlerengine/doc.go` grep for clean-review handling: for finding 5; not in context.
3. `fabricengine.ConfigDirs`: to check where a sandbox hub's committed `loom.yaml` is read from (board versus records layer); not in context.
4. A grep for `HubWide` in `internal/loomengine`: to check whether `loom.yaml` is hub-wide and so ignores a records-layer copy; it returned nothing.
5. `internal/configengine` `HubWide` and `Records` semantics: a follow-up to call 4. `loom.yaml` is not hub-wide, so D5's records-layer commit applies and holds.
