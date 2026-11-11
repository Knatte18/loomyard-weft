```yaml
lens: code-affected-set
outcome: reported
```

**Lens: code-affected-set.** Per change, the set I found and whether it is covered:

1. **`ErrCwdOutsideAnchor` text changed to "cwd is not at the recorded fabric anchor".** Covered.
   - A repo-wide grep for the old text ("outside the recorded fabric anchor" / "anchor subtree") finds nothing outside `_lym/` and `.lym/`.
   - The consumers, `internal/driftcli/cli.go:262`, `internal/preflight/predicates.go:109` and `internal/lymcwd/gate_test.go:82`, match with `errors.Is`.
   - `internal/webstercli/cli_tmux_test.go:59` asserts only that the sentinel name is absent from output.
   - The sentinel's doc comment in `internal/lymcwd/anchor.go:86` is updated and still true for the strict gate.
   - `docs/shared-libs/lymcwd.md:54` ("not exactly the anchored directory") still describes `Resolve` correctly.

2. **New `lymcwd.ResolveWithin`, and `anchorRelOrRoot` shared by three resolvers.** Covered.
   - `docs/overview.md`, `docs/shared-libs/lymcwd.md` and `pattern/PATTERN-cwd-resolution.md` all list it.
   - The new `weftname` import in `lymcwd.go` is inside the closed import set `PATTERN-cwd-resolution` names (stdlib, `dotgit`, `lymdirs`, `weftname`).
   - The `NamedWorktree` and `resolveCore` refactors keep their documented behavior, including the `"."` fallback.
   - Nothing outside the diff needs to change for the other `ResolveWorktree` and `NamedWorktree` callers (beamcli, fabriccli, hubgeom, fabricengine), because their behavior is unchanged.

3. **`await-forks` resolves through `ResolveWithin` in `internal/shuttlecli/cli.go`; new `awaitForksVerb` constant.** Covered.
   - The verb's own `Long` help text and the `cli.go` header are updated.
   - No docs or stencils state a cwd restriction for `await-forks`. My doc, spec, stencil and pattern grep for `await-forks` near "cwd", "anchor" or "director" returned only research data files (`docs/research/**/calls.tsv`), which are not an affected set.
   - `burlerengine/doc.go` and `twilldirective/doc.go` mention the verb only as a wait.

4. **Declared PATTERN changes.** The rendered constraints carry no declared-changes list, so there is nothing to land. The `PATTERN-cwd-resolution.md` background addition is consistent with the entry's one-line text.

5. **Commit messages and `Pattern-Departure:` trailers.** None are present and none are needed.

**Findings:** none.

**Tool calls:**
1. Grep for `await-forks` near cwd/anchor wording in docs, specs, stencils and pattern, plus an inventory of `ResolveWorktree` users. I needed to know whether any doc or stencil states a cwd restriction for the verb.
2. Grep for enforcement tests that pin `lymcwd.Resolve` callers, plus `docs/shared-libs/lymcwd.md` lines 40-70. My context lacked that doc section and any enforcement test; the grep found no such test.
