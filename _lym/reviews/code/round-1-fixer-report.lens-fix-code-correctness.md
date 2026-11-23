```yaml
lens: fix-code-correctness
outcome: reported
```

No findings.

Run-through:
- `viewProjects` on the fixture: the open walk emits P1, P2 and P4 with `done=false`. The done loop then emits P3 (parent P0, `done=true`) and P5 (parent P3, because `projectHandle` finds gamma by slug, `done=true`). This matches the test's expected rows and order, and the gate passed.
- The only callers of `add` are the two closures inside `viewProjects`, and both pass the new argument.
- `board.html:589`: `project.done` is `undefined` when absent and the JSON always carries `done`. The `return` inside `forEach` skips the done project without breaking the loop. `projectSelect` serves both forms, so the create and edit dropdowns are both covered.
- The tree builder at `board.html:403` still reads every project, done ones included, so the done projects still appear in the tree.
- Defect-in-place check:
  - Hard-coding the root handle at `view.go:192` makes P5's `Parent` "P0" instead of "P3", so the new row fails.
  - Dropping `Done: true` fails the P3 and P5 rows.
  - No Go test covers the HTML skip, which the review did not ask for.
- `viewFixture` users are `viewer_test.go:27` (page bytes only) and `view_test.go`. The entry map is unaffected, because P5 is a project and not an entry. The `replay_test.go:1024` row only names the file.

Calls made:
1. grep for `ViewProject`, `viewFixture()` and `projects` users in `internal/` and `cmd/`: my context lacked the other callers and consumers of the changed symbols.
