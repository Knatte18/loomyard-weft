```yaml
lens: code-affected-set
outcome: reported
```

Task: lens code-affected-set, listing per change the set found and whether it is covered.

**Per change**

1. **`ResolveWithAnchor` and `resolveWithAnchorCore` removed.**
   - Set found: a repo-wide grep, case-insensitive and excluding `.git`, `_lym` and `.lym`, finds no reference to either name. That covers code, tests, `docs/`, `PATTERN.md`, `pattern/` and the stencils.
   - The `gate_test.go` comments and both docs are updated.
   - Covered.
2. **`configengine.md` headings and layout.**
   - Set found: the grep finds no remaining "Migration from old format" and no link to the old anchors `#exported-functions` or `#migration-from-old-format`.
   - Covered.
3. **`render.go` header and recovery-prompt doc.**
   - Set found: `gateTiersOrNone` returns "none" for a blank value (`render.go:607-613`), which matches the new header.
   - `webster-prefix-recovery.md:26` reads `_lym/plan/00-overview.md`, which matches the new doc.
   - The grep finds no other `plan/overview.md` mention in the repo.
   - Covered.
4. **`adoptcli/doc.go` mark order.**
   - Set found: `primeMark` in `internal/adoptcli/check.go:350-377` puts `own.Text` first, then the other marks in `os.ReadDir` order, which is sorted by name. The doc's own `primeMark` comment at `check.go:347` agrees.
   - The new `doc.go` wording is correct.
   - Covered.
5. **`cmd/lym` tier claims.**
   - Set found: `crosscompile_test.go:28` and `main_integration_test.go` call `tiergate.Require(t, 1)`, and `main_integration_test.go` seeds repos with `gitkit.Git`. This matches the new comments.
   - The `main_test.go` tests at tier 0 and the `exitcode_test.go` and `destructiveguard_test.go` claims also match the code.
   - Covered.
6. **Stencil paths under `{{.specs_dir}}/loom/`.**
   - Set found: the grep lists many stencil uses. `stencilstore.go:259-261` confirms `RelPath` gives `<family>/<name>.md`, with the family taken from the part of the name before its first "-".
   - A spec named `loom-plan-spec` therefore deploys under `loom/`, and the summary's claim holds.
   - The diff leaves the stencils unchanged, which is consistent with that.
   - Covered.
7. **PATTERN, commit messages and departures.**
   - The diff touches no `PATTERN.md` or `pattern/` file, and the directive declares no changes.
   - I found no `Pattern-Departure:` trailers, and none are needed.
   - Covered.

**Findings:** none.

**Tool calls (1):**
1. A grep for the stale names and anchors, plus `check.go:345-380`. The manifest lacked both the repo-wide residual-reference search and the `primeMark` definition.
