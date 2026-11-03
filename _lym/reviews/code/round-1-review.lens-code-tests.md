```yaml
lens: code-tests
outcome: reported
```

Task: code-tests lens. List, per behaviour the diff adds, the test that pins it, or none.

**Per-behaviour list**

- New "Making a beam" section in `contracts/orchrole/orch-role.md:64-83`. This is static prose in a role stencil, which the lens exempts. The diff changes no Go code, no marker and no output shape.
- Role file still carries its two markers (`command_index`, `decision_note_title`). The diff adds no marker or template action. `contracts/stencils/markers_test.go:36` pins the marker set and still passes. Covered.
- Role file still registered as the fixed stencil `orch-role`. `contracts/stencils/registry_test.go:20` pins this and is unaffected by the new prose. Covered.
- Role file still embedded and rendered. `contracts/orchrole/orchrole.go` embeds the file and `internal/orchengine/prompt.go` renders it. The diff adds no template syntax (`{{`), so the render path is unchanged. Covered by the existing render and marker tests.
- Commands and flags the prose names: `lym board upsert`, `lym beam create`, `--anchor`, `lym beam start`, `lym batten status`, `lym batten cancel --remove` and `lym fabric add`. The prose adds no behavior. A test asserting the prose against the command surface would be a new assertion on a stencil's static prose, which the test-economy rule flags. None is expected.

**Findings**

None. The diff adds no behavior that needs a test row; the only added content is static stencil prose, which the lens exempts. Adding an assertion on the new section's wording would itself be a test-economy finding.

I made no tool calls: the manifest already held the diff, the marker and registry test references, and the embed and render path.
