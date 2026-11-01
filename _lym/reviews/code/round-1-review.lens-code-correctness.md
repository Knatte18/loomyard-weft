```yaml
lens: code-correctness
outcome: reported
```

**Lens: code-correctness. Per-function list, then findings.**

- `RenderRoleFile(path, index)` (`prompt.go:52`): case `index == ""`. `stencil.Fill` is handed both markers and the role renders. The result is the same as before, because the old path also filled an empty string. Correct.
- `RenderRoleFile`, role text: the file has exactly the two markers (`command_index`, `decision_note_title`) and no other `{{`, and the filled-in `<slug>` is plain text. The old missing-marker refusal is gone. That is safe, because the embedded bytes are fixed and `markers_test.go` pins the marker set. Correct.
- `writeFilled` (`prompt.go:128`):
  - `fill` fails: it returns before any directory or file is created, so no file is left behind. Correct.
  - `MkdirAll` fails: wrapped with the name, nothing written. Correct.
  - `AtomicWriteBytes` fails: wrapped with the name. Correct.
  - The `fill(..., false)` call returns the output untrimmed, matching the old `render(..., false)`. Correct.
- `renderFile` (`prompt.go:119`): read failure is wrapped as `orch: read <name>: %w`, the same message as before. The note-template path is unchanged apart from the split. Correct.
- `render` and `fill`: the single-line trim and newline check moved verbatim into `fill`. Case: multi-line override on a pointer stencil. It still errors, naming the stencil. Correct.
- Callers (`start.go:580`, `resumecontext.go:27`, `watcher.go:193`): all pass the new two-argument form. A render error is wrapped as before. `ResumeContext` and `startOrch` still pass `stencilsDir` for the other stencils. Correct.
- `watcher_test.go:1751-1752`:
  - The "index source" row now has no `stencil`, and the test takes the `indexErr != nil` branch before any `WriteFile`. So the empty stencil name is never used. Correct.
  - The removed "role" row had no code behind it any more, since the role can't be overridden. Correct.
- `registry_test.go` `shippedStencils`: a name in both `Registry()` and `fixedStencils` would be silently overwritten. There is no overlap, because `orch-role` left the registry. Correct.
- `citation_enforcement_test.go`: names now come from `slices.Sorted(maps.Keys(texts))`, so the order is deterministic, and the `RequireFloor` call is kept. Correct.
- `parentdirective_test.go` and `roleskills_test.go`: they iterate `shippedStencils`, so the orch role is scanned. Correct.
- `govalues_test.go`: the role row is keyed `orch-role` and reads from `shippedStencils`. Correct.
- `seam_enforcement_test.go`: `contracts/orchrole` is added to the import allowlist. The package imports only `embed`. Correct.
- `orchrole.go`: `//go:embed orch-role.md` sits directly above `var Role []byte`, with a blank `//` line in between that is allowed. The file exists in the same directory. Correct.

**Findings:** none.

Notes outside the lens, for the handler:
- `contracts/orchrole/orch-role.md` header (line 1) says "the same for every orch in every repository" while the role body still says "hub's prime worktree" and "hub orchestrator". That is a wording matter for the focus bullets, not a correctness defect.
- `docs/research/*` still cites the old `orch-template-role.md` path. Those are dated research snapshots, so I am not raising them.

Tool calls:
1. Read `internal/orchengine/prompt.go` lines 40-190: the diff showed only hunks, and I needed the bodies of `writeFilled`, `render` and `fill`.
2. Read `internal/orchengine/watcher_test.go` lines 1735-1805: I needed to check that the "index source" row still works with no `stencil` field.
