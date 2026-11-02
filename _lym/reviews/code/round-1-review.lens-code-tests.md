```yaml
lens: code-tests
outcome: reported
```

Lens code-tests: for each behaviour the diff adds or changes, the test row that would fail if it broke.

**Per behaviour: the test that pins it**

1. **The prompt carries the implementer PATTERN directive rendered from `WorktreeRoot`.** Pinned by the new subtest "a worktree with a PATTERN overview carries the implementer directive" (`internal/samplerengine/spec_test.go:61-83`).
   - It writes `PATTERN.md` into a temp worktree.
   - It computes the expected text with `pattern.Directive(in.WorktreeRoot, stencilsDir, pattern.RoleImplementer, nil)`.
   - It asserts the text is non-empty (the `PATTERN-example` check), so the final `strings.Contains(spec.Prompt, directive)` cannot pass vacuously.
   - Each of these breakages fails it:
     - the call is dropped;
     - the marker is removed from the stencil;
     - the wrong root is passed (`StencilsDir` instead of `WorktreeRoot`);
     - a different role is passed, because the expected text is built with `RoleImplementer` and each role reads its own stencil.

2. **A worktree with no PATTERN overview still builds, with an empty directive and no unfilled-marker error.**
   - Pinned implicitly by "a valid call builds the interactive session spec" (`spec_test.go:35-59`). Its `WorktreeRoot` is `/work/try-the-thing`, which does not exist, so `pattern.Directive` returns `("", nil)`.
   - If `stencil.FillOptional` reverted to `stencil.Fill`, or `patternDirectiveMarker` left the optional list, that subtest would fail with "unfilled top-level marker(s): pattern_directive".
   - Nothing asserts that the rendered prompt is clean of a stray `{{` or an orphan blank block in the empty case. The stencil's prose is exempt, so this is not a finding.

3. **A `pattern.Directive` failure is returned wrapped as `sampler: SessionSpec: ...`** (`spec.go:81-84`). Covered by none.
   - No row makes `pattern.Directive` fail, for example an active `PATTERN.md` with a missing directive stencil, or a stat error.
   - Severity LOW, class `design`.
   - Evidence: the new subtest and the valid subtest are the only two rows that reach this call, and both succeed.
   - Mitigation: the branch is a one-line pass-through, and the failure itself is pinned inside `internal/pattern`. I raise it only because the rubric's "behaviour the diff adds with no coverage" bullet names it.

4. **Marker set of `sampler-template-agent` now includes `pattern_directive`, placed before `## Start`.**
   - Not pinned: `contracts/stencils/markers_test.go` (`TestStencils_TopLevelMarkers`, lines 24-68) has no sampler row, and `TestStencils_MarkerPositions` has none either.
   - The sampler stencil had no row before this diff, so this is not a regression and not a finding.
   - Placement is static prose, which is exempt.
   - The marker's presence is pinned by behaviour 1.

**Findings**

- LOW, `design`: no test makes `pattern.Directive` fail inside `SessionSpec`, so the new error branch at `internal/samplerengine/spec.go:81-84` is unexercised. Evidence: `internal/samplerengine/spec_test.go:35-83`, the only two rows that reach the call.

**Redundancy and economy**

- No duplicate test.
- No review-fix test added for a non-gap finding.
- No assertion on a stencil's static prose.
- The new subtest's check that the directive contains `PATTERN-example` (`spec_test.go:77-79`) re-asserts `pattern.Directive`'s own behaviour. It is a guard against the `Contains` check passing on an empty directive, not a redundancy finding.

**Tool calls:** none. The manifest already held the diff, `spec_test.go`, `spec.go`, `pattern.go` and `markers_test.go`.
