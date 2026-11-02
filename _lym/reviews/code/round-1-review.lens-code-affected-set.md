```yaml
lens: code-affected-set
outcome: reported
```

Lens: code-affected-set. This is what the sampler change forces outside the lines it touches. I found no defects.

**Per change, the set found**

1. **`SessionSpec` renders `pattern.Directive` with `pattern.RoleImplementer`.** Covered.
   - There is one production caller, `internal/loomcli/wiring.go:933-939`. It passes `websterGeom.StencilsDir` and `location.WorktreePath()`. Both are what `Directive` reads from and what the task worktree is.
   - Argument order matches `driftengine/seats.go:225` and `darnengine/spec.go:94`, including the trailing `nil`.
   - `Directive` returns `("", nil)` for an absent `PATTERN.md` (`internal/pattern/pattern.go:99-129`), so the optional marker holds.
   - The existing valid case in `spec_test.go` uses the nonexistent worktree `/work/try-the-thing`. That is an absent overview, so it covers the no-overview path.

2. **The stencil gains `{{.pattern_directive}}`.** Covered.
   - The marker sits after the opening paragraph and before `## Start`. It stays a top-level substitution with no conditionals.
   - The banner's marker list and its "paths only, bar the PATTERN overview" wording match the code.
   - No test pins the sampler stencil's marker set. `contracts/stencils/markers_test.go` has no sampler row, and a grep of `contracts`, `internal/stencilstore` and `cmd` finds no other sampler stencil assertion. No row needs updating.
   - The stencil is registered once, in `contracts/stencils/stencils.go:48,518,637`. The diff has no second copy to update.
   - `FillOptional` tolerates the extra key for a seeded older copy of the stencil that lacks the marker.

3. **Docs.**
   - `internal/samplerengine/doc.go` is updated and matches the code.
   - `internal/pattern/doc.go` names no consumer list for the roles, so it needs no edit.
   - `docs/overview.md` lines 262-275, 526-532 and 606 describe the sampler recipe and its session. They do not describe the stencil's markers or directives, so none is stale.
   - No `PATTERN.md` entry or `pattern/` background file describes the sampler's prompt. `PATTERN-stencil-ownership` and `PATTERN-parent-directive` still hold. The stencil still renders the parent and edit directives and reads at call time through `stencilstore`.

4. **Declared PATTERN changes.** The directive declares none, and the diff touches neither `PATTERN.md` nor `pattern/`. Nothing to land.

5. **Commit message and departures.**
   - The one commit's body matches the code: `RoleImplementer`, task worktree, optional marker.
   - It carries no `Pattern-Departure:` trailer, and none is needed.

6. **Stencil wording.**
   - The diff only adds the directive and edits the banner.
   - No existing sampler rule is dropped, weakened or contradicted.

Findings: none.

Tool calls (3):
1. `grep` in `internal/pattern/doc.go`, `pattern/`, `PATTERN.md` for role and engine consumer lists, plus a grep for `samplerengine.SessionSpec` and `SessionInputs` callers and a sampler search in `docs`. The manifest did not hold the callers or the docs.
2. `grep -A8` of `SessionInputs{` outside `samplerengine`, to read what `internal/loomcli/wiring.go` passes. The manifest lacked it.
3. `grep` of `docs/overview.md` for sampler lines and of `internal/pattern/doc.go` for PATTERN lines, to check whether either describes the stencil's markers or lists the roles. The manifest lacked both.
