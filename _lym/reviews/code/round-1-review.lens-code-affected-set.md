```yaml
lens: code-affected-set
outcome: reported
```

**Lens: code-affected-set.** For each change, the set of things it touches outside its own lines, and whether each is covered.

1. **Event moved after the module-config commit in `CloneAndWire`.**
   - Set: `fabricengine/doc.go:520-533` (updated), the `CloneAndWire` doc comment (updated), and the `fabriccli/clone.go` inline comments (updated). `add.go:723` is the other `NotifyWorktreeMade` caller, and the diff leaves it alone. A `NotifyWorktreeMade` call exists nowhere else.
   - Not covered: nothing in the set is wrong.
   - The clone-time "made" assertion in `TestWorktreeSeam_Events` (`worktreeseam_integration_test.go:170-177`) still holds, because it checks event count, root, Bolt checkout present, exclude lines seeded and no paths. Moving the call after the config commit breaks none of those.

2. **`CloneResult.SeamWarning` (new field).**
   - Set: the only constructor is `CloneHub`'s return literal, which correctly leaves it empty (`fabricengine/clone.go:413-424`). The only caller of `CloneAndWire` outside tests is `runCloneWithReset`, and `hubforge` is the other caller. A grep of non-test Go found no other construction site.
   - Covered.

3. **`seam_warning` envelope field on `lym fabric clone`.**
   - Set: the `runCloneWithReset` doc comment (updated) and `fabriccli/doc.go` (updated). The `seam_warning` set on `fabric pull` (`weft_verbs.go:460-474`) is a separate envelope; its doc says "present only when…". The clone's key is "always present", as its own comment says.
   - Tests: `cli_test.go:136` asserts only `warp_binding_recorded`, and `envelope_test.go:57` builds a fixture map by hand. Neither enumerates the full clone key set, so neither goes stale.
   - Docs: a grep of `docs/`, `contracts/`, `README.md` and `plugins/` found no spec or reference that lists the clone envelope's keys, so no doc is left to update.
   - Covered.

4. **`hubforge.registerTeardown` → `RegisterTeardown`.**
   - Set: `hub.go:266,282`, `hub_test.go:416`, `template.go:235`, `hubforge/doc.go`, and the new test. A grep of `internal` and `cmd` found no leftover lowercase reference.
   - Two historical hits remain, in `docs/research/loom/plan-split/.../round-1-review-wv-plan.md:87` and `plandag-wv-split.md`. Both are dated research records and are not subjects.
   - The doc comment opening was updated to `RegisterTeardown installs…`. Covered.

5. **`hubforge/doc.go` export list.** The list now names the junction teardown. No PATTERN entry restates the export list.

6. **PATTERN entries.**
   - `PATTERN-fabric-boundary`: the fabric-isolation clause is untouched, because the seam is already general and the diff adds no heddle knowledge to fabric. Fine.
   - `PATTERN-test-speed`: the new subtest sits in the existing `tiergate.Require(t, 1)` test, so the tier rule holds. The test is not parallel, and the file header gives the reason, so that is fine.
   - `PATTERN-test-isolation`: `hubforge` is used through `fabriccli.CloneAndWire`, so that is fine.
   - The commits carry no `Pattern-Departure:` trailer, and none is needed. The recipe lists no declared PATTERN changes for this diff, and none landed.

7. **Commit `b536821a` dropping the made-event module-config assertion.** A grep for the dropped assertion's names found nothing stale. The test header and `seamHeal` comments are unaffected.

8. **Commit messages against the diff.**
   - `187419c7` says the event fires "once the module configs are committed, before the hub walk". The code matches that.
   - It says the warning is "printed as the clone envelope's `seam_warning`, naming `lym heddle render` in the prime". The text comes from `seamWarning`, which names `l.WorktreePath()` and `seamWayForward` ("run `lym heddle render` in that worktree"). The prime's path is in it, so the claim holds.

**Findings:** none. No member of the affected set is left wrong.

Tool calls (4):
1. `grep` for clone envelope key names and `registerTeardown`; this context had not searched them.
2. `grep` of `envelope_test.go` and `cli_test.go`, `internal` and `cmd` and docs, to confirm no key-set assertion, doc or old-name hit; this context had not searched them.
3. `grep` for `warp_binding_recorded` in the two tests, and `sed` of `add.go:715-730`; this context lacked the `add.go` call site and its handling.
4. `sed` of `envelope_test.go:40-75`, to confirm the clone fixture is hand-built and not a full key set.
