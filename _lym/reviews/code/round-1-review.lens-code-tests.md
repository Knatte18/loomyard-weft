```yaml
lens: code-tests
outcome: reported
```

**Per behaviour, the test that pins it, or none**

1. **`RenderRoleFile` renders the embedded role and fills both markers.** Pinned by the `role` row of `TestRenderFiles_WriteWithoutLeadingComment` (`internal/orchengine/prompt_test.go:36`). It asserts the index, `Decision for <slug>:` and the leading comment dropped.
2. **The embedded role declares `command_index` and `decision_note_title`.** Pinned by the `orch-role` row of `TestStencils_TopLevelMarkers` (`contracts/stencils/markers_test.go:36`). This replaces the deleted `TestRenderRoleFile_RefusalsWriteNoFile`, whose refusal path no longer exists, since a shipped file cannot be operator-edited.
3. **The role is outside the registry, so no Bolt copy is seeded.** Pinned by none directly.
   - The only tests that mention `orch-role` are the `fixedStencils` helper in `registry_test.go` and the marker and go-value rows.
   - `TestStencils_NameGoValues` fails if `orch-role` is also registered.
   - Re-adding an `orch-template-role` registry entry would still pass the registry-versus-disk test if the embed target were also restored.
   - This is a minor gap for the entry's central promise of no board copy a hub can edit.
4. **Shipped-stencil scans now include the role.**
   - Citation: `scanBareCitations` through `shippedStencils`.
   - Skill-load: `TestStencils_NoSkillLoadInstructions`.
   - Parent directive: `registeredBodies`.
   - Go values: `TestStencils_NameGoValues` `orch-role`/`MessageTail`.
   - All are pinned. Dropping `fixedStencils` from `shippedStencils` would not fail anything except the marker and go-value rows, which read `orchrole.Role` or `shippedStencils`.
5. **The watcher aborts the cycle when the command index fails.** Pinned by the `index source` row of `TestWatcher_FailingRenderAbortsWithoutClear` (`watcher_test.go:1751`). The `role` row was removed because an embedded role cannot fail to fill. No uncovered behaviour remains.
6. **`renderFile` and `writeFilled` refactor, and the `fill` split.** Pinned by the existing note, start and pointer render tests, including `TestRenderPointers_MultiLineOverrideFails` and `TestRenderPointers_OneLineNamingTheirPaths`. Not a finding.
7. **`RenderRoleFile` signature without `stencilsDir`.** The callers are `start.go:580`, `resumecontext.go:27` and `watcher.go:193`. The compiler pins the signature, and `ResumeContext` and watcher tests exercise the render. Not a finding.
8. **`orchengine` may import `contracts/orchrole`.** Pinned by `orchengineAllowedImports` in `seam_enforcement_test.go`, which the diff updated.
9. **The role text changes** (run-level calls settled by the orch, no status polling, rules moved in). This is static stencil prose and exempt.
10. **`orchrole` imports nothing** (claimed in `orchrole.go`'s doc comment, though it imports `embed`). No leaf-import test is cited. The package is trivial and the claim is stdlib-only, so not a finding.

**Findings**

- **LOW, `consistency`**
  - **Location:** `contracts/stencils/registry_test.go:21`, `contracts/stencils/markers_test.go:36`.
  - **Summary:** No test pins that the orch role stays out of the registry. A re-registered `orch-template-role` would re-seed an editable Bolt copy and no test would fail.
  - **Evidence:** the grep above shows no test names `orch-template-role` or asserts its absence from `Registry().Names()`.
  - **Fix:** add one assertion to the existing registry test that `Registry().Default` has no `orch-template-role` entry. Do not add a new test.

No test duplicates existing coverage, and no new assertion pins stencil prose. The `registry_test.go` helper only enumerates texts.

**Tool calls (1 of 6)**
1. Grep of `*_test.go` for `orch-template-role`, `"orch-role"` and `fixedStencils`: to check whether a test pins the registry exclusion, which my context lacked.
