```yaml
lens: code-tests
outcome: reported
```

**Per-behaviour list**

1. **Failed made event never fails the clone, and its warning reaches `CloneResult.SeamWarning` naming the prime and `lym heddle render`.**
   Pinned by the new subtest at `worktreeseam_integration_test.go:206-220`. It would fail if the call were dropped, if `res` were wiped, or if the warning text lost the prime path or the verb.

2. **The made event fires after the module-config commit.** This is the Goal's candidate cause. **No test pins it.**
   - The only clone made-event assertions are at `worktreeseam_integration_test.go:170-178`. They check root, Bolt checkout present, exclude lines seeded and no paths.
   - Commit b536821a removed the module-config assertion. Moving the call back above `ReconcileAll` and `CommitAnchoredPaths` would leave every test green.

3. **The made event fires before the hub walk, so a walk failure still leaves the prime rendered.** **No test pins it.**
   - No test makes `hubreconcile.Ensure` fail and checks that a made event was heard.
   - Moving the call after the walk would pass every test.

4. **A successful clone prints an empty `seam_warning`.** **No test pins it.**
   - The new subtest covers only the failing seam.
   - The healthy-seam clone at `:170-178` never reads `res.SeamWarning`.
   - A stuck or wrongly populated `SeamWarning` on success would pass.

5. **The `lym fabric clone` envelope carries the `seam_warning` field.** **No test pins it.**
   - `runCloneWithReset` at `internal/fabriccli/clone.go:245-254` adds the field.
   - The grep found no clone test that asserts any envelope field, `seam_warning` included. The only envelope assertion is the pull one at `worktreeseam_integration_test.go:303`.
   - The new subtest calls `CloneAndWire`, which bypasses `runCloneWithReset`. Dropping `"seam_warning": res.SeamWarning` from the map would pass.
   - The Goal asks for the failed render to be printed, and the field is how it is printed.

6. **`hubforge.RegisterTeardown` is exported.**
   - Pinned by `hub_test.go:416`, which calls it on a hand-cloned hub.
   - Also exercised by the new subtest at `worktreeseam_integration_test.go:219`.
   - This is a rename only, so there is nothing new to cover.

**Findings**

- **MEDIUM, design.** `worktreeseam_integration_test.go:170-178`: no test pins the ordering the change exists to make. The Goal's "wanted" outcome is a rendered prime. Nothing checks that the made event fires after the module-config commit or before the walk. Evidence: the assertions at `:170-178` and the diff of b536821a. Fix: add an ordering assertion to the clone made-event subtest. Record the weft HEAD or the committed module-config state when the made event fires, and check it is the post-commit state. Add a walk-failure row that checks the event was heard before the clone failed.
- **MEDIUM, design.** `internal/fabriccli/clone.go:253`: the `seam_warning` clone envelope field has no test. Evidence: `runCloneWithReset` at `:245-254`; grep over `internal/fabriccli` and `internal/fabricengine` tests. Fix: add a clone-envelope row, for example next to the pull envelope check at `worktreeseam_integration_test.go:303`. Drive the `lym fabric clone` verb with a failing seam and assert `"seam_warning"` and `lym heddle render` in the output.
- **LOW, design.** `worktreeseam_integration_test.go:170-178`: the healthy clone's empty `SeamWarning` is not asserted. Fix: add `res.SeamWarning == ""` to the made-event subtest. This needs the clone result, which `hubforge.NewHub` does not expose, so the check may fold into the new subtest as a second clone with the seam healed.

**Tool calls**
1. `grep` over `internal/fabriccli` and `internal/fabricengine` tests for `seam_warning` and `"warning"`, and a list of fabriccli test files naming clone. My context held no clone-envelope test.
