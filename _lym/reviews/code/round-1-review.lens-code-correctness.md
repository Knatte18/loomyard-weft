```yaml
lens: code-correctness
outcome: reported
```

Lens: code-correctness. I ran each changed function and branch by hand against the template and the reconcile code in context. I made no tool calls.

**Functions and branches, with the case run:**

1. **`template.yaml` with no layer file → `Load` → `decode` → `decodeDefaults`.**
   With no layer present, `Load` decodes the template (`internal/runprofile/runprofile.go:169-173`). The defaults are tapestry, motif, darn, pawl and raddle. Each names a profile that holds that recipe's block, so `decodeDefaults` passes. Correct.
2. **`Default("arras")` on the template.**
   `Defaults` holds no `arras` key, and `heldFormerName` finds no rename whose `to` is arras (`runprofile.go:108,148-156`). It returns `"", nil`. Correct.
3. **`Settings(p, "arras")` on each starting profile.**
   `profile["arras"]` is absent and arras has no former name, so it returns the "holds no arras block" error with its way forward (`runprofile.go:135-141`). Correct.
4. **Seed-Child `prepareRunSettings` for an arras task naming no profile.**
   `Default` returns `""`, so the task's settings stand alone (`internal/battencli/runsettings.go:34-43`). Correct, and the new `doc.go` line describes this.
5. **Seed-Child for an arras task naming `solid`.**
   The `Settings` error is wrapped as `ErrInvalidRunSettings` and the run goes Stuck with a way forward (`internal/battencli/runsettings.go:44-48`). The refusal is correct and actionable. The board write path checks only `Lookup` (`internal/boardcli/runsettings.go:46-53`), so the board still accepts that entry. That is unchanged behaviour.
6. **`writeRunUsage` for an arras run with no default.**
   `Default` returns `""` and the usage record's profile is empty (`internal/loomcli/sharedbootstrap.go:217-225`). Correct.
7. **Raddle context.**
   `raddlecontext.go` reads only the tapestry and darn coverage, so arras is not involved (`internal/loomcli/raddlecontext.go:114-121`). Correct.
8. **Hub reconcile, untouched file with a base present.**
   Every arras unit equals the base and the template lacks it, so `followUnit` removes it. The emptied mappings drop with it (`internal/yamlengine/follow.go:113-121,172-175`). The result loads. Correct.
9. **Hub reconcile where the operator repointed `defaults.arras` to `light` (or `wide`, `speed`) and left the arras blocks untouched.**
   - `defaults.arras` differs from the base value `solid`, so it is kept as the operator's (`follow.go:164-169`).
   - `light.arras` equals the base, so it is removed (`follow.go:172-175`).
   - The written file holds `defaults: {arras: light}` with no `light.arras` block.
   - `decodeDefaults` then fails with "recipe "arras" names profile "light", which holds no arras block" (`runprofile.go:251-253`), and `Load` returns `ErrInvalid` for the whole file.
   - Neither caller checks the load after writing: `ReconcileHubWideAt` has no load check (`internal/configsync/configsync.go:632-643`), and neither do the hub reconcile at `internal/hubreconcile/walk.go:250-265` or `internal/configcli/hubwide.go:224-232`. The broken file is committed.
   - Every consumer that loads profiles then fails, for every recipe, not only arras:
     - Seed-Child (`internal/battencli/runsettings.go:26-29`)
     - the board's profile validation (`internal/boardcli/runsettings.go:47-50`)
     - the raddle context (`internal/loomcli/raddlecontext.go:58-61`)
     - `checkProfiles` (`internal/configreg/runsettings.go:231-235`)
   - Defect.
10. **Hub reconcile where the operator edited one key of `solid.arras` (for example `split.max_bounces: 5`) and left `defaults.arras` at `solid`.**
    - `defaults.arras` equals the base, so it is removed.
    - The `solid.arras` mapping is descended: `discussion` is removed, `split.max_bounces` is kept and `split.hard_max_bounces` is removed.
    - What remains is `solid.arras.loom.split: {max_bounces: 5}`, which is still a mapping of mappings, so `decodeProfile` passes.
    - Arras has no default. The leftover partial block loads, but it is no longer the full block and has no default naming it. That is consistent with "operator values stay".
    - Correct.
11. **Hub reconcile where the operator added profile `mine` with an arras block and set `defaults.arras: mine`.**
    Both are kept, `mine.arras` exists and the file loads. Correct.
12. **Hub reconcile with no template base.**
    The `haveBase=false` branch never removes anything (`follow.go:156-165`). The arras keys stay and the file loads, so the keys are not dropped. That is expected behaviour for a file with no base.

**Findings:**

- **HIGH (BLOCKING), design. Location: `internal/runprofile/template.yaml:22` (the removed `arras: solid` default together with the removed arras blocks).**
  When an operator has repointed `defaults.arras` to another starting profile and left that profile's arras block untouched, the hub reconcile keeps the default and drops the block. It then commits a `profiles.yaml` that `runprofile.Load` refuses. That refusal breaks Seed-Child, the board's profile writes, the raddle context and the config load check for every recipe on the hub, not only for arras.
  Evidence: `internal/yamlengine/follow.go:164-175`, `internal/runprofile/runprofile.go:251-253`, `internal/configsync/configsync.go:632-643` (no load check after the write), `internal/hubreconcile/walk.go:250-265`.
  No test row covers the case. The summary's claim that "an arras key the operator changed stays, as for any other template change" holds key by key but omits this cross-key invariant.

Tool calls: none.
