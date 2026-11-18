```yaml
lens: code-correctness
outcome: reported
```

I reviewed the code-correctness lens. The diff changes no production code; the only changed code branch is in the test.

- `editdirective.Directive` is not changed by the diff (`internal/editdirective/editdirective.go:23-30`). Each engine's real shipped copy goes through the same read, strip-leading-comment and trim-trailing-newline steps as before, and the new lines 18–20 carry no marker. Correct.
- `TestDirective_ReadsTheEnginesCopyWithoutBanner`, the new `scratchBuildBan` branch (`internal/editdirective/editdirective_test.go:20-21,41-43`):
  - **Case run:** `claude` and `codex`, each seeded by `stencilkit.Seed`. `Seed` reconciles `stencils.Registry()` in `ModeProduction` (`internal/testkit/stencilkit/stencilkit.go:26-39`), so `Directive` returns the shipped copies.
  - **Text match:** joined, the const reads "never build, restore or compile anything under them, and never create a git worktree, clone or checkout there." That is byte-identical to line 12 of both diffs (`002-…claude.diff:12`, `003-…codex.diff:12`), each a single line with no break inside the phrase. `strings.Contains` holds for both engines. Correct.
  - **Failure case:** if either copy dropped or reworded the ban, `Contains` would be false and `t.Errorf` would name the engine, the text it got and the const. That error is reported, not swallowed. Correct.
  - **Error path:** when `Directive` fails, line 36 still calls `t.Fatalf` before the new check runs, so no nil or empty string reaches it. Correct.
- Prose in `internal/editdirective/doc.go:16-19`, `contracts/orchrole/orch-role.md`, and both `edit-directive.md` copies is not code, so this lens has nothing to run there.

Findings: none.

Tool calls:
1. `grep` of `stencilkit.Seed`/`SeedInto` and `editdirective.Directive`. My context did not show where the seeded stencils come from, which decides whether the test reads the shipped copies, or `Directive`'s body.
