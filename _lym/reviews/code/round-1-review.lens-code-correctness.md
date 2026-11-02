```yaml
lens: code-correctness
outcome: reported
```

Task: run the code-correctness lens over `SessionSpec`, the stencil edit and the new test.

**Per function, branch and case (all correct unless marked)**

1. `SessionSpec`, required-input loop, `WorktreeRoot` = "": refuses by name before any directive work, so `pattern.Directive`'s empty-root early return is never reached. Correct.
2. `SessionSpec`, the call `pattern.Directive(in.WorktreeRoot, in.StencilsDir, pattern.RoleImplementer, nil)` against `VerifyTable` (`internal/driftengine/seats.go:225`) and `DarnTable` (`internal/darnengine/spec.go:94`):
   - Arguments match: worktree root, stencils dir, role, `nil`.
   - The call comes after the stencil read and the parent and edit directives, and a failure returns a zero `Spec` wrapped as `sampler: SessionSpec: %w`. Nothing is left half-built. Correct.
3. Worktree path that does not exist, as in the existing valid case (`/work/try-the-thing`): `readOverview` gets `os.IsNotExist` and returns `""`. `Directive` returns `("", nil)`, and `FillOptional` renders the marker as nothing, so the prompt still builds. Correct.
4. `PATTERN.md` absent, a directory, or whitespace-only: each returns `("", nil)`. Correct.
5. A stat or read failure other than not-exist (EACCES, or ENOTDIR when a path component is a file): `SessionSpec` fails with `sampler: SessionSpec: pattern: stat …`. This is the same fail-informative behaviour as `DarnTable` and `VerifyTable`. Correct.
6. Active overview:
   - `Directive` returns the filled implementer text.
   - `FillOptional` passes the value through `Execute` without re-parsing it, so a `{{` inside the overview cannot be interpreted as a template action. The value is not HTML-escaped (text/template).
   - Correct.
7. `stencil.FillOptional` with `[]string{patternDirectiveMarker}`: the optional marker is exempt from the unfilled guard and a whitespace-only value is normalised to `""`. The other five markers stay required, so an empty `parent_directive` or `edit_directive` still fails loudly. The stencil has no `{{if}}`, so the branch-internal guard is not involved. Correct.
8. Stencil placement of `{{.pattern_directive}}` between the role paragraph and `## Start`: with an empty directive the output has extra blank lines between the paragraph and `## Start`. This changes no instruction, so I raise no finding. Correct.
9. Task-worktree claim: the one production caller passes `location.WorktreePath()` as `WorktreeRoot` (`internal/loomcli/wiring.go:937`), the same value the stencil names as `task_worktree`. So the directive reads `PATTERN.md` from the task worktree, and the commit body and doc claim hold. Correct.
10. New subtest, "a worktree with a PATTERN overview carries the implementer directive":
    - It writes a `PATTERN.md` into `t.TempDir()` and calls `SessionSpec`.
    - It computes the expected directive with the same `Directive` call and asserts the overview's entry is in it, then that the prompt contains the directive.
    - If `SessionSpec` dropped the marker or filled it empty, the final assertion fails. `t.TempDir()` and `t.Parallel` do not conflict.
    - Correct.

**Findings:** none.

**Tool calls**
1. `grep` for `SessionInputs` and `WorktreeRoot:` callers: the manifest did not hold the production caller of `SessionSpec`, which the worktree-root claim depends on.
2. `Read` of `internal/loomcli/wiring.go` lines 915–949: to see what the caller passes as `WorktreeRoot`.
