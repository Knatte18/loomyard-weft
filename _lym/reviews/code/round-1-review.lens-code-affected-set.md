```yaml
lens: code-affected-set
outcome: reported
```

code-affected-set lens: what the diff's changes force outside the lines it touches. I found one finding.

**Per change, the set found and whether the diff covers it**

1. `fabricengine.ConfigSpec`: `branch_prefix` goes from CategoryCrossPair to CategoryMachine.
   - `configengine.MachineKeys` and `LayerFile` (layers.go:146-149): the machine layer now stacks for fabric whenever `Dirs` carries both Anchor and Board. Covered.
   - `fabricengine.ConfigDirs` (junctionnames.go:130) always sets Anchor, so every `LoadConfig(ConfigDirs(...))` call site reads the machine value with no caller change. That includes the `fabriccli` verbs, `beamcli/target.go:99`, `beamcli/deps.go:37`, `pairparts.go:66` and `anchordeps.go:287`. Covered.
   - The `Dirs{Board}`-only loads in `configcli/hubwide.go:329-402` are hub-file checks. They correctly skip the machine layer. Covered.
   - `configcli` `selectedLayer` (hubwide.go:117-120): fabric no longer meets the "no machine key" refusal, so `--layer machine` reaches `LayerFile`. Covered by generic code. The refusal test row moves to slack, which is hub-wide with one CategoryNone key (slackengine/config.go:41-43), so its `wantErr` "lym config slack" still holds. Covered.
   - `ValidateLocal` (layers.go:337-339): a local `branch_prefix` is still refused, now as "is machine" instead of "is cross-pair". No doc or test asserts the old text for this key. Covered.
   - `configreg_test.go` `wantMachineKeys` has a fabric row. Covered.
   - Readers that turn a branch into a slug with the current prefix: `slugOfBranch` callers in cleanup.go:265, cleanupremotewarp.go:164 and 215, and pairparts.go:78 (`TaskBranchSlugs`, read by `adoptcli/fixloop.go:326,396` and `beamcli/deps.go:280`). **Not covered.** See finding 1.
2. Wording of the CategoryMachine definition (configengine/layers.go, configengine/doc.go, pattern/PATTERN-config-layers.md).
   - These three agree. PATTERN.md's one-liner for `PATTERN-config-layers` doesn't describe the categories and needs no edit. `configcli/doc.go:39` ("holds a value every worktree of the hub on this machine shares") and `configreg/doc.go:11-13` still read true. Covered.
3. Fabric docs: template.yaml comment, fabricengine/doc.go:767-768, docs/overview.md:519-520.
   - There is no other list of machine keys in docs or overview, and no remaining "fabric has no machine key" or "every pair shares" claim about fabric (grep). The `LYM_BRANCH_PREFIX` env reference is unaffected. Covered.
4. Commits and declared PATTERN changes: neither commit carries a `Pattern-Departure:` trailer, and the directive declares no PATTERN changes. Nothing to check.

**Findings**

1. MEDIUM, design. Location: `internal/fabricengine/pairbranch.go:29` (`slugOfBranch`) and its callers `internal/fabricengine/pairparts.go:65-79`, `internal/fabricengine/cleanupremotewarp.go:164,215` and `internal/fabricengine/cleanup.go:265`.
   - Summary: once prefixes differ per machine, every step that turns a branch into a slug strips only this machine's prefix. A pair made on another machine (`a/foo`, read on a machine with prefix `b/`) then gets the slug `a/foo`.
   - Effects:
     - `TaskBranchSlugs`, which adopt's fix loop and beam read, lists `a/foo` as a slug.
     - The orphan and remote-warp sweeps take the teardown lock `a/foo.lock` (teardownlock.go:43-44). That is not the `foo.lock` a same-machine teardown of the adopted pair holds, so the two are not serialized.
     - `keptReason` checks weft evidence under the wrong slug.
   - Before this diff, a mismatch needed a hub-wide prefix change. Now two machines with different prefixes on one origin hit it routinely.
   - The diff touches none of these readers, and the change description's Open risks names only `PairTaskBranchOrOrigin`.
   - Evidence: pairbranch.go:29-31, pairparts.go:60-79, cleanupremotewarp.go:164-166 and 215-217, cleanup.go:265-271, `_lym/landing/summary.md` "Open risks".

**Tool calls**

1. A grep for `branch_prefix`, branch-prefix and cross-pair wording across .md, .yaml and .go: the manifest's doc grep did not cover comments in other fabricengine files or tools/sandbox.
2. A read of `ValidateLocal`'s refusal message in configengine/layers.go: the manifest lacked the local-refusal text a machine key now produces.
3. A grep of docs/overview.md and README for `fabric.yaml` and `LYM_BRANCH_PREFIX`: to find other descriptions of fabric's keys the manifest lacked.
